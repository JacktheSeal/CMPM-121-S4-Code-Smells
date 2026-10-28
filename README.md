# CMPM-121-S4-Code-Smells
Group Members: Tyler Roth, Jack Seales, Alekzander Matthews

# Code Smell 1: Duplicated Code
## Introduction:
Duplicate code occurs when identical or nearly identical code structures appear across multiple parts of a codebase, typically arising from the copy and pasting your code or a separate developer implementing it in parallel. It can be very problematic as it drastically increases maintenance overhead, requiring bug fixes to be replicated everywhere and heightening the risk of lingering inconsistencies.
## Prevention and refactoring:
Prevent duplicate code by conducting thorough code reviews regularly. To fix existing duplicates, you can us-e Fowler's refactoring patterns such as Extract Function for shared local logic, Slide Statements to align matching code blocks, and Pull Up Method to elevate duplicated subclass logic into a superclass.
## Code examples:
### Python:
Here is a example in Python of bad code exhibiting this smell:
```python
def defeat_goblin(player):
    xp_gained = 50
    if player.has_double_xp_potion:
        xp_gained = 50 * 2
    player.score += xp_gained

def defeat_dragon(player):
    xp_gained = 500
    if player.has_double_xp_potion:
        xp_gained = 500 * 2
    player.score += xp_gained
```
This code is problematic as the double XP potion check is copy pasted into both enemy defeat functions. If game designers decide to change the potion bonus to triple XP, they'd have to hunt down and update every enemy function in the codebase. Comparing this to if we refactor the code to the following:
```python
def add_xp(player, base_xp):
    multiplier = 2 if player.has_double_xp_potion else 1
    player.score += base_xp * multiplier

def defeat_goblin(player):
    add_xp(player, 50)

def defeat_dragon(player):
    add_xp(player, 500)
```
The xp potion check is controlled entirely in the new add_xp function which makes updating the code in the future a much less difficult process that only needs to be done in one place.
### JavaScript:
Here is a example in JavaScript of bad code exhibiting this smell:
```JavaScript
function completeQuest(player, quest) {
    let earned = quest.reward;
    if (player.isVip) {
        earned = earned * 1.5;
    }
    player.xp += earned;
    console.log("Popup: You got " + earned + " XP!");
}

function defeatBoss(player, boss) {
    let earned = boss.reward;
    if (player.isVip) {
        earned = earned * 1.5;
    }
    player.xp += earned;
    console.log("Popup: You got " + earned + " XP!");
}
```
In this example the VIP reward bonus code as well as the reward popup are duplicated in both the quest completion code and the boss defeating code. If a future designer decides to change the game to not be full of evil micro transactions later down the line, you'd have to change the code in both of these functions (and probably more). To fix this you can refactor the code like this:
```JavaScript
function grantXp(player, baseAmount) {
    let earned = player.isVip ? baseAmount * 1.5 : baseAmount;
    player.xp += earned;
    console.log("Popup: You got " + earned + " XP!");
}

function completeQuest(player, quest) {
    grantXp(player, quest.reward);
}

function defeatBoss(player, boss) {
    grantXp(player, boss.reward);
}
```
Using the Extract Function so that now all the logic for gaining XP is handled in grantXp(), which is then called by other functions. This allows for the easy and logical change of removing the micro transactions. 

# Code Smell 2: Long Function

## Introduction
Long functions often get created over the course of development through small adjustments and development pressures. They often form because developers make many small changes without thinking about where they are going, and end up shoving them all into one function. This is bad because it reduces other developer's ability to understand the code properly, as well as 

## Prevention and Refactoring
It can be difficult to prevent long functions from occurring, especially when you have a short or rushed development cycle. However, the best way to prevent long functions is to think about your code's structure before you write anything. That way, you'll prevent unnecessary lengthening of functions, and help to maintain a cleaner, more readable codebase without the need for as much refactoring. However, when looking to refactor long functions, the best solution is almost always to find a way to use Extract Function. If the opportunity arises, things like Replace Temp with Query and Introduce Parameter Object might also help.
## Code Examples

### Example 1: JavaScript
```JavaScript
update(time, delta) {
        let my = this.my;
        const speed = 2;  // pixels per second
        let dt = speed * (delta / 1000);  // convert delta from ms to seconds, and multiply by speed to get pixels/tick

        //Check for completion
        if (
            !this.levelComplete &&
            this.currentWave >= LEVELS.level1.length &&
            this.enemies.getLength() === 0
        ) {
            this.levelComplete = true;

            this.showLevelCompleteMessage();
        }

        //Wave Spawning
        this.waveTimer+= delta;

        if (this.waveActive && this.enemies.getLength() === 0) {
            this.waveActive = false;
        }

        const levelData = LEVELS.level1;

        if (
            !this.waveActive &&
            !this.levelComplete &&
            this.currentWave < levelData.length
        ) {
            this.spawnWave(levelData[this.currentWave]);
            this.waveActive = true;
            this.currentWave++;
        }

        //MOVEMENT

        // Moving left
        if (this.left.isDown) {
            // Check to make sure the sprite can actually move left
            if (my.sprite.player.x > (my.sprite.player.displayWidth/2)) {
                my.sprite.player.x -= this.playerSpeed * dt;
            }
        }

        // Moving right
        if (this.right.isDown) {
            // Check to make sure the sprite can actually move right
            if (my.sprite.player.x < (this.game.config.width - (my.sprite.player.displayWidth/2))) {
                my.sprite.player.x += this.playerSpeed * dt;
            }
        }


        //BULLET SPAWNING
        for (let enemy of this.enemies.getChildren()) {

            enemy.shootTimer++;

            if (enemy.shootTimer >= enemy.shootCooldown) {

                enemy.shootTimer = 0;
                this.shootSound.play();
                this.spawnEnemyBullet(enemy);
            }
        }

        // Player Bullet
        if (Phaser.Input.Keyboard.JustDown(this.space)) {
            // Are we under our bullet quota?
            this.playerShootSound.play();

            if (this.my.sprite.bullet.getLength() >= this.maxBullets) {
                return; // don't spawn a new bullet
            }
            const bullet = this.physics.add.sprite(
                my.sprite.player.x,
                my.sprite.player.y - my.sprite.player.displayHeight / 2,
                "player_bullet"
            ).setScale(4);

            bullet.body.setSize(
                bullet.width * 0.25,
                bullet.height * 0.25
            );

            bullet.body.setOffset(
                bullet.width * 0.375,
                bullet.height * 0.375
            );

            bullet.body.setCollideWorldBounds(true);
            bullet.body.onWorldBounds = true;

            bullet.body.setVelocityY(-300);
            this.my.sprite.bullet.add(bullet);
        }


        this.physics.world.on("worldbounds", (body) => {
            body.gameObject.destroy();
        });




        //COLLISION CHECKS
    }
```
This code is clearly problematic due to its length. A good rule of thumb is to keep your update function as clean and readable as possible, and this clearly does not do that. In order to clean up this code, we actually already have some comments which give us clear places to extract functions. For example, by extracting completeion, wave spawning, movement, and bullet checks, the code would look something more like this. Bear in mind not all the 
```JavaScript
update(time, delta) {
        let my = this.my;
        const speed = 2;  // pixels per second
        let dt = speed * (delta / 1000);  // convert delta from ms to seconds, and multiply by speed to get pixels/tick

        checkCompletion();
        spawnWave(time, delta);
        doMovement();
        spawnEnemyBullets();
        spawnPlayerBullets();
    }
```
As you can see, by moving all of the code into separate functions, the update function becomes much more readable and comprehensive.

## Example 2: GDScript (Godot)
```GDScript
func show_dialogue(lines: Array[String], npc_portrait: Texture2D, name: String, scale : float):
	talking = true
	panel.show()
	
	portrait.texture = npc_portrait
	portrait.scale = Vector2(scale, scale)
	name_label.text = name
	
	while Input.is_action_pressed("interact"):
		await get_tree().process_frame

	for line in lines:
		play_random_meow()
		await type_text(line)

		# Wait for player to press interact
        while !Input.is_action_just_pressed("interact"):
            await get_tree().process_frame

		# Wait until player releases interact
		while Input.is_action_pressed("interact"):
			await get_tree().process_frame
			
	label.text = ""
	panel.hide()
	talking = false
```
This code is particularly poor because it lacks comments explaining what each part of the function is doing. When you have a lot of work but a short development cycle, sometimes readability can be recovered by using comments, as looking for those comments can act almost like seeing function names if organized properly. However, without any comments nor helper functions, the code becomes bloated. One again, extracting functions will help this code look much more readable.
```GDScript
func show_dialogue(lines: Array[String], npc_portrait: Texture2D, name: String, scale : float):
	talking = true

    displayPortrait()
    processDialogue()
    hidePortrait()
```
As you can see, this is much more readable.
