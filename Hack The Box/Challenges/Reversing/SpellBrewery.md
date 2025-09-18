# SpellBrewery

## Recon

ELF is a wrapper around DLL.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_spellbrewery]
└─$ file SpellBrewery.dll
SpellBrewery.dll: PE32 executable for MS Windows 4.00 (console), Intel i386 Mono/.Net assembly, 3 sections
```

Run:

```plaintext
ds
```

## Reversing

```c#
private static void BrewSpell() {
    bool flag = Brewery.recipe.Count < 1;

    if (flag) {
        Console.WriteLine("You can't brew with an empty cauldron");
    } else {
        byte[] array = Brewery.recipe.Select((Ingredient ing) => (byte)(Array.IndexOf<string>(Brewery.IngredientNames, ing.ToString()) + 32)).ToArray<byte>();
        bool flag2 = Brewery.recipe.SequenceEqual(Brewery.correct.Select((string name) => new Ingredient(name)));
        if (flag2) {
            Console.WriteLine("The spell is complete - your flag is: " + Encoding.ASCII.GetString(array));
            Environment.Exit(0);
        } else {
            Console.WriteLine("The cauldron bubbles as your ingredients melt away. Try another recipe.");
        }
    }
}

private static readonly string[] correct = new string[] {
    "Phantom Firefly Wing", "Ghastly Gourd", "Hocus Pocus Powder", "Spider Sling Silk", "Goblin's Gold", "Wraith's Tear", "Werewolf Whisker", "Ghoulish Goblet", "Cursed Skull", "Dragon's Scale Shimmer",
    "Raven Feather", "Dragon's Scale Shimmer", "Ghoulish Goblet", "Cursed Skull", "Raven Feather", "Spectral Spectacles", "Dragon's Scale Shimmer", "Haunted Hay Bale", "Wraith's Tear", "Zombie Zest Zest",
    "Serpent Scale", "Wraith's Tear", "Cursed Crypt Key", "Dragon's Scale Shimmer", "Salamander's Tail", "Raven Feather", "Wolfsbane", "Frankenstein's Lab Liquid", "Zombie Zest Zest", "Cursed Skull",
    "Ghoulish Goblet", "Dragon's Scale Shimmer", "Cursed Crypt Key", "Wraith's Tear", "Black Cat's Meow", "Wraith Whisper"
};
```

Correct ingredients are:

- `Phantom Firefly Wing`
- `Ghastly Gourd`
- `Hocus Pocus Powder`
- `Spider Sling Silk`
- `Goblin's Gold`
- `Wraith's Tear`
- `Werewolf Whisker`
- `Ghoulish Goblet`
- `Cursed Skull`
- `Dragon's Scale Shimmer`
- `Raven Feather`
- `Dragon's Scale Shimmer`
- `Ghoulish Goblet`
- `Cursed Skull`
- `Raven Feather`
- `Spectral Spectacles`
- `Dragon's Scale Shimmer`
- `Haunted Hay Bale`
- `Wraith's Tear`
- `Zombie Zest Zest`
- `Serpent Scale`
- `Wraith's Tear`
- `Cursed Crypt Key`
- `Dragon's Scale Shimmer`
- `Salamander's Tail`
- `Raven Feather`
- `Wolfsbane`
- `Frankenstein's Lab Liquid`
- `Zombie Zest Zest`
- `Cursed Skull`
- `Ghoulish Goblet`
- `Dragon's Scale Shimmer`
- `Cursed Crypt Key`
- `Wraith's Tear`
- `Black Cat's Meow`
- `Wraith Whisper`

```plaintext
d
```
