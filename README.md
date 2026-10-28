# CMPM-121-S4-Code-Smells
Group Members: Tyler Roth, Jack Seales, Alekzander Matthews

# Code Smell 1: Duplicated Code
## introduction:
Duplicate code occurs when identical or nearly identical code structures appear across multiple parts of a codebase, typically arising from the copy and pasting your code or a seperate devloper implementing it in parallel. It can be very problematic as it drastically increases maintenance overhead, requiring bug fixes to be replicated everywhere and heightening the risk of lingering inconsistencies.
## prevention and refactoring:
Prevent duplicate code by conducting thorough code reviews regularly. To fix existing duplicates, you can use Fowler's refactoring patterns such as Extract Function for shared local logic, Slide Statements to align matching code blocks, and Pull Up Method to elevate duplicated subclass logic into a superclass.
## Code examples:
### Python:
Here is a example in python of bad code exhibiting this smell:
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
The xp potion check is controlled entirely in the new add_xp function which makes updating the code in the future a much less dificult process that only needs to be done in one place.
### JavaScript:
Here is a example in javascript of bad code exhibiting this smell:
```javascript
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
In this exampel the VIP reward bonus code as well as the reward popup are dumpicated in both the quest completion code and the boss defeating code. If a future designer decide to change the game to not be full of evil micro transactions later down the line, you'd have to change the code in both of these funtions (and probably more). To fix this you can refactor the code like this:
```javascript
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
Using the Extract Function so that now all the coe for gaining the XP is contorlled in a single function that is called by the other ones. Allowing for the easy and logical change of removing the microtransations. 

# Code Smell 2: Long Function
- blah
# Code Smell 3: Comments
- bwah
