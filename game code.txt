#include "raylib.h"
#include "raymath.h" 
#include <vector>
#include <stddef.h>
#include <stdlib.h>


const int screenWidth = 1800;
const int screenHeight = 1000;

const int scarfySpeed = 10;
 double gravity = 0.9;
const int groundYPos = (3 * screenHeight) / 4;
const int numStones = 3; 
const int numCoins = 4;  
int numDeath = 0;
int health = 100;
const int numhearts = 1;
const int jumpUpFrame = 3;
const int jumpDownFrame = 4;
const int footstepFrame1 = 1;
const int footstepFrame2 = 4;
bool sl = false;
bool tryb = true;
const int numchest = 1;


bool isFootstepFrame(int frameIndex) {
    return (frameIndex == footstepFrame1 || frameIndex == footstepFrame2);
}

bool isScarfyOnGround(Texture2D* scarfy, Vector2* scarfyPosition) {
    return (scarfyPosition->y + scarfy->height >= groundYPos);
}

bool isTextureValid(const Texture2D* texture) {
    return texture->id > 0;
}

void showErrorAndExit(const char* errMsg) {
    while (!WindowShouldClose()) {
        BeginDrawing();
        ClearBackground(RAYWHITE);
        DrawText(errMsg, 20, 20, 20, BLACK);
        EndDrawing();
    }
    exit(EXIT_FAILURE);
}

void showCantLoadFileErrorAndExit(const char* filename) {
    showErrorAndExit(TextFormat("ERROR: Couldn't load %s.", filename));
}

void cleanup() {
    CloseAudioDevice();
    CloseWindow(); 
}

bool checkCollision(Rectangle rect1, Rectangle rect2) {
    return (rect1.x < rect2.x + rect2.width &&
        rect1.x + rect1.width > rect2.x &&
        rect1.y < rect2.y + rect2.height &&
        rect1.y + rect1.height > rect2.y);
}

int main(void) {
    InitWindow(screenWidth, screenHeight, "Scarfy Bhaag");
    InitAudioDevice();
    atexit(cleanup);

    // Load start button image
    const char* startButtonFile = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\playbutton.png";
    Texture2D startButton = LoadTexture(startButtonFile);
    if (!isTextureValid(&startButton)) {
        showCantLoadFileErrorAndExit(startButtonFile);
    }
    // Load logo button image
    const char* logoButtonFile = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\logo.png";
    Texture2D logoButton = LoadTexture(logoButtonFile);
    if (!isTextureValid(&logoButton)) {
        showCantLoadFileErrorAndExit(logoButtonFile);
    }

    // Load won button image
    const char* wonButtonFile = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\won.png";
    Texture2D wonButton = LoadTexture(wonButtonFile);
    if (!isTextureValid(&wonButton)) {
        showCantLoadFileErrorAndExit(wonButtonFile);
    }
    // Load lost button image
    const char* lostButtonFile = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\gope.png";
    Texture2D lostButton = LoadTexture(lostButtonFile);
    if (!isTextureValid(&lostButton)) {
        showCantLoadFileErrorAndExit(lostButtonFile);
    }
    // Load lost button image
    const char* tryyButtonFile = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\scorecard.png";
    Texture2D tryyButton = LoadTexture(tryyButtonFile);
    if (!isTextureValid(&tryyButton)) {
        showCantLoadFileErrorAndExit(tryyButtonFile);
    }
    // Load background image
    const char* bgFilename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bc21.jpg";
    Texture2D background = LoadTexture(bgFilename);
    if (!isTextureValid(&background)) {
        showCantLoadFileErrorAndExit(bgFilename);
    }
    // Load background image for new level
    const char* bg2Filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bc8.jpg";
    Texture2D background2 = LoadTexture(bg2Filename);
    if (!isTextureValid(&background2)) {
        showCantLoadFileErrorAndExit(bg2Filename);
    }
    // Load background image for new level 3
    const char* bg3Filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bc22.jpg";
    Texture2D background3 = LoadTexture(bg3Filename);
    if (!isTextureValid(&background3)) {
        showCantLoadFileErrorAndExit(bg3Filename);
    }
    // Load background image for new level 4
    const char* bg4Filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bc24.jpg";
    Texture2D background4 = LoadTexture(bg4Filename);
    if (!isTextureValid(&background4)) {
        showCantLoadFileErrorAndExit(bg3Filename);
    }
    //load font file
    Font customFont = LoadFont("C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\Venite1.ttf");
    Font customFont2 = LoadFont("C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\Venite2.ttf");

    // Load scarfy texture
    const char* filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\scarfyfull.png";
    Texture2D scarfy = LoadTexture(filename);
    if (!isTextureValid(&scarfy)) {
        showCantLoadFileErrorAndExit(filename);
    }

    unsigned numFrames = 6;
    int frameWidth = scarfy.width / numFrames;
    Rectangle frameRec = { 0.0f, 0.0f, (float)frameWidth, (float)scarfy.height };
    Vector2 scarfyPosition = { float(screenWidth / 2.0f), groundYPos - float(scarfy.height) };
    Vector2 scarfyVelocity = { 0.0f, 0.0f };

    // Load sound effects
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\a.mp3";
    Sound footstepSound = LoadSound(filename);
    if (!footstepSound.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\b.mp3";
    Sound landingSound = LoadSound(filename);
    if (!landingSound.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    //coin receiving sound
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\received.mp3";
    Sound receiving = LoadSound(filename);
    if (!receiving.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    //hitting sound
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\hurt3.mp3";
    Sound hurt = LoadSound(filename);
    if (!hurt.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    //gameover sound
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\gameover.mp3";
    Sound gameover = LoadSound(filename);
    if (!gameover.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    //heart sound

    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\heartsound1.mp3";
    Sound heartsound = LoadSound(filename);
    if (!heartsound.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    // jump sound
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\jump1.mp3";
    Sound jumpsound = LoadSound(filename);
    if (!heartsound.frameCount) {
        showCantLoadFileErrorAndExit(filename);
    }
    // Load background music
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bellaciao2.mp3"; // Replace with your music file path
    Music bgMusic = LoadMusicStream(filename);
    if (!bgMusic.ctxData) {
        showCantLoadFileErrorAndExit(filename);
    }
    PlayMusicStream(bgMusic);  

    unsigned frameDelay = 5;
    unsigned frameDelayCounter = 0;
    unsigned frameIndex = 0;

    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\stone.png";
    Texture2D stone = LoadTexture(filename);
    if (!isTextureValid(&stone)) {
        showCantLoadFileErrorAndExit(filename);
    }
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\stone_hitbox.png";  //  stone image
    Texture2D stone_hitbox = LoadTexture(filename);
    if (!isTextureValid(&stone_hitbox)) {
        showCantLoadFileErrorAndExit(filename);
    }
    // Load coin texture
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\coin3.png";  //  coin image path
    Texture2D coin = LoadTexture(filename);
    if (!isTextureValid(&coin)) {
        showCantLoadFileErrorAndExit(filename);
    }

    //load death coin
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\death.png";  // death coin image path
    Texture2D death = LoadTexture(filename);
    if (!isTextureValid(&death)) {
        showCantLoadFileErrorAndExit(filename);
    }
    // load heart coin
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\heart2.png";  // heart image path
    Texture2D heart = LoadTexture(filename);
    if (!isTextureValid(&heart)) {
        showCantLoadFileErrorAndExit(filename);
    }
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\scorecard.png";
    Texture2D sc = LoadTexture(filename);
    if (!isTextureValid(&sc)) {
        showCantLoadFileErrorAndExit(filename);
    }


    //load danger texture
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\danger.png";
    Texture2D danger = LoadTexture(filename);
    if (!isTextureValid(&danger)) {
        showCantLoadFileErrorAndExit(filename);
    }

    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bc20.jpg";
    Texture2D bc20 = LoadTexture(filename);
    if (!isTextureValid(&bc20)) {
        showCantLoadFileErrorAndExit(filename);
    }

    //final level
    filename = "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\bc19.jpg";
    Texture2D bc19 = LoadTexture(filename);
    if (!isTextureValid(&bc19)) {
        showCantLoadFileErrorAndExit(filename);
    }
    // chest loading
    filename= "C:\\Users\\Sheikh Hamza\\Desktop\\game resources\\chest.png";
    Texture2D chest = LoadTexture(filename);
    if (!isTextureValid(&chest)) {
        showCantLoadFileErrorAndExit(filename);
    }

    
    Vector2 stonePositions[numStones];
    Vector2 stoneVelocities[7];
    Vector2 stone_hitboxPositions[numStones];
    Vector2 stone_hitboxVelocities[numStones];
    for (int i = 0; i < numStones; i++) {
        stonePositions[i].x = (float)(GetRandomValue(0, screenWidth - stone.width));
        stonePositions[i].y = -stone.height;  // Start above the screen
        stoneVelocities[i].y = 2 + GetRandomValue(0, 3);  // Random falling speed
    }
    int score = 0;
    int level = 1;
    bool go = false;
    bool bscarfy = false;

    
    Vector2 coinPositions[numCoins];
    Vector2 coinVelocities[numCoins];
    for (int i = 0; i < numCoins; i++) {
        coinPositions[i].x = -coin.width;  // Start off the screen on the left
        coinPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650)); // Random vertical position
        coinVelocities[i].x = 4 + GetRandomValue(0, 2);  // Random horizontal speed
    }

    Vector2 heartPositions[numhearts];
    Vector2 heartVelocities[numhearts];
    for (int i = 0; i < numhearts; i++) {
        heartPositions[i].x = -heart.width;  // Start off the screen on the left
        heartPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));
        heartVelocities[i].x = 4 + GetRandomValue(0, 2);  // Random horizontal speed
    }


    Vector2 chestPositions[numchest];
    Vector2 chestVelocities[numchest];
    for (int i = 0; i < numchest; i++) {
        chestPositions[i].x = -chest.width;  // Start off the screen on the left
        chestPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650)); // Random vertical position
        chestVelocities[i].x = 3 + GetRandomValue(0, 2);  // Random horizontal speed
    }



    std::vector<Vector2> deathPositions(numDeath);  // Vector to hold positions
    std::vector<Vector2> deathVelocities(numDeath);  // Vector to hold velocities

    for (int i = 0; i < numDeath; i++) {
        deathPositions[i].x = -death.width;  // Start off the screen on the left
        deathPositions[i].y = GetRandomValue(50, screenHeight - 300);  // Random vertical position
        deathVelocities[i].x = 4 + GetRandomValue(0, 2);  // Random horizontal speed
    }

    bool gameStarted = false; 
    bool gameFrozen = true;  
    SetTargetFPS(60);

    while (!WindowShouldClose()) {
        UpdateMusicStream(bgMusic);  // Keep the music stream updated

        if (!gameFrozen) {
           
            if (isScarfyOnGround(&scarfy, &scarfyPosition)) {
                if (IsKeyDown(KEY_UP) || IsGamepadButtonDown(0, GAMEPAD_BUTTON_LEFT_FACE_UP) || IsKeyDown(KEY_SPACE)) {
                    scarfyVelocity.y = -2 * scarfySpeed;
                    PlaySound(jumpsound);
                }
            }

            if (!isScarfyOnGround(&scarfy, &scarfyPosition)) {
                if (IsKeyDown(KEY_DOWN)) {
                    gravity = 5;
                  //  PlaySound(jumpsound);
                }
            }
            else {
                gravity = 0.9;
            }
            // Handle horizontal movement (left/right) independently
            if (IsKeyDown(KEY_RIGHT) || IsGamepadButtonDown(0, GAMEPAD_BUTTON_LEFT_FACE_RIGHT)) {
                if (scarfyPosition.x < 1680) { // Prevent moving out of bounds
                    scarfyVelocity.x = scarfySpeed;
                    if (frameRec.width < 0) {
                        frameRec.width = -frameRec.width;
                    }
                }
                else {
                    scarfyVelocity.x = 0;
                }
            }
            else if (IsKeyDown(KEY_LEFT) || IsGamepadButtonDown(0, GAMEPAD_BUTTON_LEFT_FACE_LEFT)) {
                if (scarfyPosition.x > 0) { // Prevent moving out of bounds
                    scarfyVelocity.x = -scarfySpeed;
                    if (frameRec.width > 0) {
                        frameRec.width = -frameRec.width;
                    }
                }
                else {
                    scarfyVelocity.x = 0;
                }
            }
            else {
                scarfyVelocity.x = 0;
            }


            bool scarfyMoving = scarfyVelocity.x != 0.0f || scarfyVelocity.y != 0.0f;
            bool wasScarfyOnGround = isScarfyOnGround(&scarfy, &scarfyPosition);
            scarfyPosition = Vector2Add(scarfyPosition, scarfyVelocity);
            bool scarfyIsOnGround = isScarfyOnGround(&scarfy, &scarfyPosition);
            if (scarfyIsOnGround) {
                scarfyVelocity.y = 0;
                scarfyPosition.y = groundYPos - scarfy.height;
                if (!wasScarfyOnGround) {
                    PlaySound(landingSound);
                }
            }
            else {
                scarfyVelocity.y += gravity;
            }

            ++frameDelayCounter;
            if (frameDelayCounter > frameDelay) {
                frameDelayCounter = 0;

                if (scarfyMoving) {
                    if (scarfyIsOnGround) {
                        ++frameIndex;
                        frameIndex %= numFrames;

                        if (isFootstepFrame(frameIndex)) {
                            PlaySound(footstepSound);
                        }
                    }
                    else {
                        if (scarfyVelocity.y < 0) {
                            frameIndex = jumpUpFrame;
                        }
                        else {
                            frameIndex = jumpDownFrame;
                        }
                    }
                    frameRec.x = (float)frameWidth * frameIndex;
                }
            }
            Rectangle scarfyBounds = { scarfyPosition.x, scarfyPosition.y, fabs(frameRec.width), scarfy.height };

            // Update stones (falling)
            for (int i = 0; i < numStones; i++) {
                stonePositions[i].y += stoneVelocities[i].y;
                stone_hitboxPositions[i].y = stonePositions[i].y;
                stone_hitboxVelocities[i].y = stoneVelocities[i].y;
                Rectangle stone_hitboxBounds = { stone_hitboxPositions[i].x = stonePositions[i].x, stone_hitboxPositions[i].y, (float)stone_hitbox.width - 9.7 , (float)stone_hitbox.height - 9.7 };
                Rectangle scarfyBounds = { scarfyPosition.x, scarfyPosition.y, fabs(frameRec.width) - 7.7, scarfy.height - 7.7 };
                //Rectangle scarfyBounds = { scarfyPosition.x, scarfyPosition.y, fabs(frameRec.width)-7.7, scarfy.height-7.7 };

                // Check collision with Scarfy
                if (checkCollision(stone_hitboxBounds, scarfyBounds)) {

                    PlaySound(hurt);
                    health--;
                    // if i want to disappear stone the these are the lines to be added
                    // stonePositions[i].x = -stone.width;
                    // stonePositions[i].y = (float)GetRandomValue(350, 650);

                    if (health == 0) {

                        StopMusicStream(bgMusic);
                        PlaySound(gameover);
                        StopSound(hurt);

                        gameFrozen = true;  // Freeze the game on collision
                        go = true;
                        tryb = true;
                    }
                }

                // If the stone goes below the ground, reset its position to the top
                if (stonePositions[i].y > screenHeight) {
                    stonePositions[i].y = -stone.height;
                    stonePositions[i].x = (float)(GetRandomValue(0, screenWidth - stone.width));
                }
            }

            // Update coins (falling from left to right)
            for (int i = 0; i < numCoins; i++) {

                coinPositions[i].x += coinVelocities[i].x;
                Rectangle coinBounds = { coinPositions[i].x, coinPositions[i].y, (float)coin.width, (float)coin.height };
                // Check collision with Scarfy
                if (checkCollision(coinBounds, scarfyBounds)) {
                    // Handle coin collection (for example, increase score)
                    coinPositions[i].x = -coin.width;  // Move coin out of screen
                    coinPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));
                    PlaySound(receiving);
                    score++;


                    

                    
                    if (score % 10 == 0 && score != 0) {
                        //level++;
                        //  stoneVelocities[i].y += 1;
                        numDeath++;  // Increase the number of death coins when score is 10 or more
                        deathPositions.resize(numDeath);  
                        deathVelocities.resize(numDeath);  

                        // Re-initialize the new death coin positions and velocities
                        deathPositions[numDeath - 1].x = -death.width;  // New death coin starts off the screen
                        deathPositions[numDeath - 1].y = GetRandomValue(50, screenHeight - 300);  // Random vertical position
                        deathVelocities[numDeath - 1].x = 4 + GetRandomValue(0, 2);  // Random horizontal speed

                    }

                    //(for right side)

                    //    for (int i = 0; i < numCoins; i++) {
                    //        // Move coins to the left
                    //        coinPositions[i].x -= coinVelocities[i].x;

                    //        // Define coin bounds for collision detection
                    //        Rectangle coinBounds = {
                    //            coinPositions[i].x,
                    //            coinPositions[i].y,
                    //            (float)coin.width,
                    //            (float)coin.height
                    //        };

                    //        // Check collision with Scarfy
                    //        if (checkCollision(coinBounds, scarfyBounds)) {
                    //            // Handle coin collection
                    //            coinPositions[i].x = screenWidth + coin.width;  // Reset coin to the right side of the screen
                    //            coinPositions[i].y = (float)GetRandomValue(350, 650);  // Random vertical position

                    //            PlaySound(receiving);  // Play sound for coin collection
                    //            score++;  // Increase the score

                    //            // Check for every 10 points to update difficulty
                    //            if (score % 10 == 0 && score != 0) {
                    //                numDeath++;  // Increase the number of death coins

                    //                // Resize vectors for new death coin
                    //                deathPositions.resize(numDeath);
                    //                deathVelocities.resize(numDeath);

                    //                // Initialize the new death coin's position and velocity
                    //                deathPositions[numDeath - 1].x = screenWidth + death.width;  // Start off-screen on the right
                    //                deathPositions[numDeath - 1].y = GetRandomValue(50, screenHeight - 300);  // Random vertical position
                    //                deathVelocities[numDeath - 1].x = 4 + GetRandomValue(0, 2);  // Random horizontal speed
                    //            }
                    //        }

                    //        // Reset coins if they move off the left side of the screen
                    //        if (coinPositions[i].x < -coin.width) {
                    //            coinPositions[i].x = screenWidth + coin.width;  // Reset to the right side of the screen
                    //            coinPositions[i].y = (float)GetRandomValue(350, 650);  // Random vertical position
                    //        }
                    //    }






                }

                // If the coin goes off the right side, reset its position
                if (coinPositions[i].x > screenWidth) {
                    coinPositions[i].x = -coin.width;
                    coinPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));  // Random vertical position
                }
            }
            // Update death coins (falling from left to right)
           // Update death coins (falling from left to right)
            for (int i = 0; i < numDeath; i++) {
                deathPositions[i].x += deathVelocities[i].x;
                Rectangle deathBounds = { deathPositions[i].x, deathPositions[i].y, (float)death.width - 20.1, (float)death.height - 20.1 };

                // Check collision with Scarfy
                if (checkCollision(deathBounds, scarfyBounds)) {
                    PlaySound(hurt);
                    health--;
                    //if i want to disappear black coins
                 //   deathPositions[i].x = -death.width;
                  //  deathPositions[i].y = (float)GetRandomValue(350, 650);

                    if (health == 0) {
                        StopMusicStream(bgMusic);

                        PlaySound(gameover);
                        StopSound(hurt);
                        gameFrozen = true;
                        go = true;

                        tryb = true;// Freeze the game on collision with a death coin
                    }
                }



                // If the death coin goes off the right side, reset its position
                if (deathPositions[i].x > screenWidth) {
                    deathPositions[i].x = -death.width;
                    deathPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));  // Random vertical position
                }
            }
            // Heart coins
            if (health < 50) {

                for (int i = 0; i < numhearts; i++) {
                    // Move hearts to the left
                    heartPositions[i].x -= heartVelocities[i].x;

                    // Define heart bounds for collision detection
                    Rectangle heartBounds = { heartPositions[i].x, heartPositions[i].y, ((float)heart.width) - 10, ((float)heart.height) - 10 };

                    // Check collision with Scarfy
                    if (checkCollision(heartBounds, scarfyBounds)) {
                        // Handle coin collection (e.g., increase health)
                        heartPositions[i].x = screenWidth + heart.width;  // Reset heart to the right side of the screen
                        heartPositions[i].y = (float)GetRandomValue(350, 650); // Random vertical position
                        PlaySound(heartsound);
                        health += 10;
                    }

                    // If the heart moves off the left side of the screen
                    if (heartPositions[i].x < -heart.width) {
                        heartPositions[i].x = screenWidth + heart.width; // Reset to the right side of the screen
                        heartPositions[i].y = (float)GetRandomValue(350, 650); // Random vertical position
                    }
                }
            }

            if ((score > 45 && score < 60)) {

                for (int i = 0; i < numchest; i++) {
                    // Move hearts to the left
                    chestPositions[i].x -= chestVelocities[i].x;

                    // Define heart bounds for collision detection
                    Rectangle chestBounds = { chestPositions[i].x, chestPositions[i].y, ((float)chest.width) - 10, ((float)chest.height) - 10 };

                    // Check collision with Scarfy
                    if (checkCollision(chestBounds, scarfyBounds)) {
                        // Handle coin collection (e.g., increase health)
                        chestPositions[i].x = screenWidth + heart.width;  
                        chestPositions[i].y = (float)GetRandomValue(350, 650); 
                        PlaySound(heartsound);
                        score += 5;
                    }

                    // If the heart moves off the left side of the screen
                    if (chestPositions[i].x < -chest.width) {
                        chestPositions[i].x = screenWidth + chest.width; 
                        chestPositions[i].y = (float)GetRandomValue(350, 650); 
                    }
                }
            }

            if (IsKeyDown(KEY_C) && IsKeyDown(KEY_F3)) {
                health += 1000;

            }

        }

        // Draw
        BeginDrawing();
        ClearBackground(RAYWHITE);

        // Draw the background
        float bgScaleX = (float)screenWidth / background.width;
        float bgScaleY = (float)screenHeight / background.height;
        float scale = fmax(bgScaleX, bgScaleY); // Ensure no distortion, scale proportionally
        float offsetX = (screenWidth - background.width * scale) / 2.0f;
        float offsetY = (screenHeight - background.height * scale) / 2.0f;
        DrawTextureEx(background, Vector2{ offsetX, offsetY }, 0.0f, scale, WHITE);

        // background2

        // Draw the background2
        if (score >= 20 && score < 40) {
            float bg2ScaleX = (float)screenWidth / background2.width;
            float bg2ScaleY = (float)screenHeight / background2.height;
            float scale = fmax(bg2ScaleX, bg2ScaleY); // Ensure no distortion, scale proportionally
            float offsetX = (screenWidth - background2.width * scale) / 2.0f;
            float offsetY = (screenHeight - background2.height * scale) / 2.0f;
            DrawTextureEx(background2, Vector2{ offsetX, offsetY }, 0.0f, scale, WHITE);
        }
        if (score >= 40 && score < 60) {
            float bg3ScaleX = (float)screenWidth / background3.width;
            float bg3ScaleY = (float)screenHeight / background3.height;
            float scale = fmax(bg3ScaleX, bg3ScaleY); // Ensure no distortion, scale proportionally
            float offsetX = (screenWidth - background3.width * scale) / 2.0f;
            float offsetY = (screenHeight - background3.height * scale) / 2.0f;
            DrawTextureEx(background3, Vector2{ offsetX, offsetY }, 0.0f, scale, WHITE);
        }
        if (score >= 60 && score < 75) {
            float bg4ScaleX = (float)screenWidth / background4.width;
            float bg4ScaleY = (float)screenHeight / background4.height;
            float scale = fmax(bg4ScaleX, bg4ScaleY); // Ensure no distortion, scale proportionally
            float offsetX = (screenWidth - background4.width * scale) / 2.0f;
            float offsetY = (screenHeight - background4.height * scale) / 2.0f;
            DrawTextureEx(background4, Vector2{ offsetX, offsetY }, 0.0f, scale, WHITE);
        }
        if (score >= 75 && score <90) {
            float bc20ScaleX = (float)screenWidth / bc20.width;
            float bc20ScaleY = (float)screenHeight / bc20.height;
            float scale = fmax(bc20ScaleX, bc20ScaleY); // Ensure no distortion, scale proportionally
            float offsetX = (screenWidth - bc20.width * scale) / 2.0f;
            float offsetY = (screenHeight - bc20.height * scale) / 2.0f;
            DrawTextureEx(bc20, Vector2{ offsetX, offsetY }, 0.0f, scale, WHITE);
        }
        if (score >= 90) {
            float bc19ScaleX = (float)screenWidth / bc19.width;
            float bc19ScaleY = (float)screenHeight / bc19.height;
            float scale = fmax(bc19ScaleX, bc19ScaleY); // Ensure no distortion, scale proportionally
            float offsetX = (screenWidth - bc19.width * scale) / 2.0f;
            float offsetY = (screenHeight - bc19.height * scale) / 2.0f;
            DrawTextureEx(bc19, Vector2{ offsetX, offsetY }, 0.0f, scale, WHITE);
        }


        if (!gameStarted) {
            // Draw the start button
            Vector2 logoButtonPosition = { screenWidth / 2 - logoButton.width / 2, (screenHeight / 2 - logoButton.height / 2) - 150 };
            DrawTexture(logoButton, logoButtonPosition.x, logoButtonPosition.y, WHITE);
            Vector2 startButtonPosition = { screenWidth / 2 - startButton.width / 2, (screenHeight / 2 - startButton.height / 2) + 150 };
            DrawTexture(startButton, startButtonPosition.x, startButtonPosition.y, WHITE);



            // Check if the start button is clicked
            if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
                Rectangle startButtonRect = { startButtonPosition.x, startButtonPosition.y, (float)startButton.width, (float)startButton.height };
                if (CheckCollisionPointRec(GetMousePosition(), startButtonRect)) {
                    gameFrozen = false;
                    bscarfy = true;
                    sl = true;
                    gameStarted = true;  // Mark the game as started
                }
            }



        }
        if (!gameFrozen) {
            if (health < 50) {
                // Draw the start button
                Vector2 dangerPosition = { screenWidth / 2 - danger.width / 2, (screenHeight / 2 - danger.height / 2) - 150 };
                DrawTexture(danger, dangerPosition.x - 775, dangerPosition.y + 300, WHITE);
            }
            // Draw scarfy character
            if (bscarfy)
                DrawTextureRec(scarfy, frameRec, scarfyPosition, WHITE);

            // Draw stones
            for (int i = 0; i < numStones; i++) {
                DrawTexture(stone, stonePositions[i].x, stonePositions[i].y, WHITE);
            }

            // Draw coins
            for (int i = 0; i < numCoins; i++) {
                DrawTexture(coin, coinPositions[i].x, coinPositions[i].y, WHITE);
            }
            //draw death coins



            for (int i = 0; i < numDeath; i++) {
                DrawTexture(death, deathPositions[i].x, deathPositions[i].y, WHITE);
            }
            //draw hearts

            for (int i = 0; i < numhearts; i++) {
                DrawTexture(heart, heartPositions[i].x, heartPositions[i].y, WHITE);

            }
            //draw chest
            if ((score > 45 && score < 60)) {
                for (int i = 0; i < numchest; i++) {
                    DrawTexture(chest, chestPositions[i].x, chestPositions[i].y, WHITE);

                }
            }

            //if (health < 50) {
            //    // Draw the start button
            //    Vector2 dangerPosition = { screenWidth / 2 - danger.width / 2, (screenHeight / 2 - danger.height / 2) - 150 };
            //    DrawTexture(danger, dangerPosition.x-775, dangerPosition.y+300, WHITE);
            //}

            // Display "Game Over" when game is frozen
            if (sl) {
                if (score < 40 || (score > 59)) {
                    Vector2 scorePosition = { 100, 35 }; // Position of the score
                    DrawTextEx(customFont, TextFormat("Score: %d", score), scorePosition, 40, 7, WHITE);
                    // Draw the health using the custom font
                    Vector2 healthPosition = { 100, 70 }; // Position for the health
                    DrawTextEx(customFont, TextFormat("Health: %d", health), healthPosition, 40, 7, WHITE);
                }

                else {
                    Vector2 scorePosition = { 100, 35 }; // Position of the score
                    DrawTextEx(customFont, TextFormat("Score: %d", score), scorePosition, 40, 7, BLACK);
                    // Draw the health using the custom font
                    Vector2 healthPosition = { 100, 70 }; // Position for the health
                    DrawTextEx(customFont, TextFormat("Health: %d", health), healthPosition, 40, 7, BLACK);
                }
                // Draw the level using the custom font
              //  Vector2 levelPosition = { 850, 78 }; // Position for the level
               // DrawTextEx(customFont, TextFormat("Level: %d", (score / 22) + 1), levelPosition, 40, 7, GRAY);
                Vector2 level2Position = { 850, 78 }; // Position for the level
                if (score <= 19) {
                    // Vector2 level2Position = { 850, 78 }; // Position for the level
                    DrawTextEx(customFont, TextFormat("Level: %d", 1), level2Position, 40, 7, WHITE);
                }

                if (score > 19 && score < 40) {
                    // Vector2 level2Position = { 850, 78 }; // Position for the level
                    DrawTextEx(customFont, TextFormat("Level: %d", 2), level2Position, 40, 7, WHITE);
                }
                if (score >= 40 && score < 60) {
                    // Vector2 level2Position = { 850, 78 }; // Position for the level
                    DrawTextEx(customFont, TextFormat("Level: %d", 3), level2Position, 40, 7, BLACK);
                }
                if (score >= 60 && score < 75) {
                    // Vector2 level2Position = { 850, 78 }; // Position for the level
                    DrawTextEx(customFont, TextFormat("Level: %d", 4), level2Position, 40, 7, WHITE);
                }
                if (score >= 75 && score<90) {
                    // Vector2 level2Position = { 850, 78 }; // Position for the level
                    DrawTextEx(customFont, TextFormat("Level: %d", 5), level2Position, 40, 7, BLACK);
                }
                if (score >= 90) {
                    // Vector2 level2Position = { 850, 78 }; // Position for the level
                    DrawTextEx(customFont, TextFormat("Final Level " ), level2Position, 40, 7, WHITE);
                }

            }
        }
        if (go) {

            //   DrawText("GAME OVER", screenWidth / 2 - 100, screenHeight / 2 - 20, 40, WHITE);
            Vector2 lostButtonPosition = { screenWidth / 2 - lostButton.width / 2, (screenHeight / 2 - lostButton.height / 2) };


            Vector2 tryyButtonPosition = { (screenWidth / 2 - tryyButton.width / 2), (screenHeight / 2 - tryyButton.height / 2) + 150 };
            Vector2 scPosition = { (screenWidth / 2 - sc.width / 2), (screenHeight / 2 - sc.height / 2) + 230 };
            if (tryb) {
                DrawTexture(tryyButton, tryyButtonPosition.x, tryyButtonPosition.y, WHITE);
                DrawTexture(lostButton, lostButtonPosition.x, lostButtonPosition.y, WHITE);
                DrawTexture(sc, scPosition.x, scPosition.y, WHITE);
                Vector2 textPos = { tryyButtonPosition.x + 30 , tryyButtonPosition.y + 20 };
                Vector2 textPos2 = { tryyButtonPosition.x + 33 , tryyButtonPosition.y + 100 };
                DrawTextEx(customFont2, TextFormat("PLAY AGAIN"), textPos, 35, 5, BLACK);
                DrawTextEx(customFont2, TextFormat("SCORE  %d", score), textPos2, 33, 5, BLACK);
            }
            if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT) || IsKeyDown(KEY_ENTER)) {
                Rectangle tryyButtonRect = { tryyButtonPosition.x, tryyButtonPosition.y, (float)tryyButton.width, (float)tryyButton.height };
                if (CheckCollisionPointRec(GetMousePosition(), tryyButtonRect)) {

                    tryb = false;
                    // Reset game variables
                    score = 0;
                    // for (int i = 0; i < 7; i++)
                         //    stoneVelocities[i].y -= (level - 1);
                    health = 100;
                    level = 1;
                    numDeath = 0;
                    PlayMusicStream(bgMusic);
                    scarfyPosition = { float(screenWidth / 2.0f), groundYPos - float(scarfy.height) };
                    scarfyVelocity = { 0.0f, 0.0f };

                    for (int i = 0; i < numStones; i++) {
                        stonePositions[i].x = (float)(GetRandomValue(0, screenWidth - stone.width));
                        stonePositions[i].y = -stone.height;
                    }

                    for (int i = 0; i < numCoins; i++) {
                        coinPositions[i].x = -coin.width;
                        coinPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));
                    }
                    for (int i = 0; i < numhearts; i++) {
                        heartPositions[i].x = -heart.width;
                        heartPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));
                    }
                    for (int i = 0; i < numchest; i++) {
                        chestPositions[i].x = -chest.width;
                        chestPositions[i].y = (float)GetRandomValue((float)GetRandomValue(350, 400), (float)GetRandomValue(600, 650));
                    }

                    deathPositions.clear();
                    deathVelocities.clear();

                    gameFrozen = false;
                    bscarfy = true;
                    sl = true;
                    gameStarted = true; // Mark the game as started
                }
            }


            // StopMusicStream(bgMusic);
          //  PlaySound(hurt);


        }
        //if (score == 95) {
        //    gameFrozen = true;
        //    Vector2 wonButtonPosition = { screenWidth / 2 - wonButton.width / 2, (screenHeight / 2 - wonButton.height / 2) };
        //    DrawTexture(wonButton, wonButtonPosition.x, wonButtonPosition.y, WHITE);
        //    // DrawText("YOU WON WOOAH!", (screenWidth / 3)+100, screenHeight / 2, 50, WHITE);
        //}

        EndDrawing();
    }
    StopMusicStream(bgMusic);

    // Cleanup resources
    UnloadMusicStream(bgMusic);
    UnloadSound(footstepSound);
    UnloadSound(jumpsound);
    UnloadSound(landingSound);
    UnloadTexture(background);
    UnloadTexture(stone);
    UnloadTexture(heart);
    UnloadTexture(scarfy);
    UnloadTexture(coin);  
    cleanup();

    return EXIT_SUCCESS;
}