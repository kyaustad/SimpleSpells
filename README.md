Here is a quick start guide to getting a custom spell in the demo map that comes with the project. There are other pages dedicated to specific topics but this is a quick rundown of adding a new effect to the existing demo map.

## Add a new effect

in the *SpellPlayground* folder under *Enums*, open the *E_SpellEffects* enum and add a new entry here for the effect you wish to add. The demo spell crafting UI will auto populate it once we add it as a craftable effect. For this quick start guide, I will add a simple 'Fortify Mana' spell effect.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/1.png)

### Add the new effect to the crafting menu

Next, under the path *SpellPlayground/Demo/UI/Widgets* open the *WBP_SpellCraftingMenu* and go the the graph tab in the upper right (not designer). There is a variable there titled *PossibleSpellEffects*. Select it and add a new element to the array and select your newly added effect.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/2.png)

### Add the Interface functions and implement them in the Spell Manager

The last two steps go hand in hand. We need to add functions for our new effect to the blueprint interface in the root folder of SpellPlayground called *BPI_SpellEffects*. Once that is open add a new effect for starting and ending. In this example I will add one titled *FortifyMana_Start* and assign it to the start category for ease of use. I will do the same for a *FortifyMana_End* and assign it to the end category. Not every spell needs an ending function, only ones where an effect needs to be removed. So damaging abilities like Fire Damage do not require an end function because there is no effect to remove. Spells like fortify, invisibility or levitate for example, do require an end function because we need to remove those effects when the spell ends.

We also need to make sure our new functions have the same signature as the others with an input variable of type *actor* and an input variable of type *float*. The easiest way to do this is to simple duplicate a function in the start category and rename it and do the same for a function in the end category.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/3.png)

Once that is done, open the *AC_SpellManager* Actor Component blueprint. Open the function *ApplySpellEffect*. Here is a simple function that switches based on the spell effect to call the correct effect on the intended target. Simply drag off the Switch node from your new effect (If it isn't visible, right click on the switch node and hit 'refresh node') and search for you new function. In this case mine is called *Fortify Mana Start* and add that function to the execution of that effect. Duplicate the *Target*, *Caster*, and *Strength* variables from the other functions and plug them in to your new function.


![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/4.png)


![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/5.png)

Once that is finished, do the same thing in the *RemoveSpellEffect* function for your newly created effect.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/6.png)

## Implement the new effect

Last thing to do is to actually make our new effect do something. So on the player pawn *BP_FirstPersonCharacter* (which can be found in *SpellPlayground/Demo/FirstPerson/Blueprints*) we need to implement our new interface function for starting and ending our effect. So with the pawn blueprint open, on the left under *Interfaces* we need to find our newly created start and end functions for our Fortify Mana spell. Double click the *FortifyMana_Start* and the *FortifyMana_End* to get an event node for those functions.


![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/7.png)


![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/8.png)

For this demo effect, it should use the included stat component for tracking player and NPC health, mana and stamina and increase the mana by the strength. So we simply need to drag a reference to the *AC_Demo_Stats* component into our graph and call an included function on that component called *Increase Max Mana*.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/9.png)

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/10.png)

Then for the Fortify Mana End function we do the same thing but call the *Decrease Max Mana* function from our demo stats component reference.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/11.png)

### Prevent Effect Stacking

Spell effects trigger each second using a custom timer component so that the event that is called when the timer triggers is unique to each spell effect. Certain effects though we do not want to re-apply every second the timer triggers. Fire damage is an example of a spell we do want that behavior because a spell that has 5pts of strength for 5 seconds duration will apply 5 points each second or tick of the timer totaling 25pts of damage. Fortify Mana or Fortify Health however we do not want to be adding the strength to our max each second. We simply want it to add the amount once and keep the amount at the new max for the duration of the spell, and then when the spell ends, reset the max to the previous value.

To solve this, in the *AC_SpellManager* component there are two variables. *SpellEffectsToResetOnReApply* and *SpellEffectsThatShouldNotTickPerTimerCount* (a mouthful I know but I wanted them to be clear). These are arrays of spell effects and have two distinctions. *SpellEffectsToResetOnReApply* simply resets the timer of that effect when the same effect is cast on a target who already has that effect, i.e. it will prevent stacking but will make the effect last the newly casted duration. *SpellEffectsThatShouldNotTickPerTimerCount* on the other hand can stack, i.e. have multiple of the same effect on a target. However they do not trigger the effect every second the timer component ticks. For example if we cast a *Summon Follower* spell and it is in the *SpellEffectsToResetOnReApply* array but not in the *SpellEffectsThatShouldNotTickPerTimerCount* then the demo implementation of the summon effect will be called each second that spell is active on the target but it will be the same effect. So the same summon spell effect will be called each second resulting in a new follower each second. If it wasn't in *SpellEffectsToResetOnReApply* but was in *SpellEffectsThatShouldNotTickPerTimerCount* then we could have multiple followers as the effect would stack but each second an effect is active it wouldn't reapply the effect, i.e. summon a new follower.

So for our example effect, Fortify Mana, we want it to be in both arrays. You could have fortify spells stack if you want by leaving them out of *SpellEffectsToResetOnReApply* but for this example I do not want Fortify Mana to stack. So add an entry to each variable with our new effect. This will prevent Fortify Mana from stacking and it will prevent the effect from being called each second the effect is active on the target.

## Adding the spell to the player's spell list

With all that done, your new spell is ready to go! The player can craft the spell at the demo spell crafting altar! Give it a try! However, if you want to manually create a spell to be in the player's spell list when the game starts, we can do that. Re-open our pawn blueprint *BP_FirstPersonCharacter* and select the *AC_SpellManager* from the list of components. In the details panel find the category called Spells and add a new element to the array variable *Owned Spells*.


![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/12.png)

Expand your newly added element and fill out the details of your spell. A name, description, icon, casting animation and add your new spell effect and select a Cast Type, duration and strength. See below for my filled out example.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/13.png)

With all that done, hit play and open the spell list and you should see your new spell. Select it and cast and double check that it works as intended.

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/14.png)

![](https://git.crabinteractive.com/crabdev/SimpleSpellsWiki/raw/branch/main/Pics/QuickStart/15.png)

## Conclusion

This should give you an idea of how to get started implementing spell effects of your own using this framework. This was heavily inspired by old school RPG's specifically Morrowind. Effects can be implemented on anything! In the demo map there is a chest, for example, that only implements the lock, and open spell effects. To get your new spell effect on the NPC's simple open the *BP_NPC_Base* class and implement the events just like you did for the player. 
