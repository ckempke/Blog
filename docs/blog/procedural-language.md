---
title: "Procedural Language Generation"
date: 2022-8-16
author: Christopher Kempke
description: Generate "decent-sounding" languages procedurally
tags: 
  - language
  - procedural generation
  - technology
  - games
---

# Procedural Language Generation for a Fantasy Game

_No Man's Sky_ has an interesting—if not particularly realistic—language mechanism.  Each of its three main races, plus the Atlas, have their own language.   The player learns these languages (if they wish), one word at a time, and when presented with text in the "alien" language, known words are substituted with the player's chosen game language (which I'll assume to be English for convenience, here, although the game ships with several real-world languages available).

This makes finding and learning new words a background game objective, and adds more things for the player "to do" that produce meaningful rewards, which is always a challenge for procedural games, where things like quests, monsters, and locations are almost literally cookie-cutter.

 NMS makes several simplifying assumptions:

- A "word" in the alien language corresponds to exactly one "word" in English. 
- Even more unrealistically:  word order, grammar, and punctuation of the alien languages correspond exactly to English, as well.   Effectively all alien languages are substitution ciphers, where you take an English sentence, substitute an alien word for each English one, and produce the result.
- Things like homographs (words that are the same spelling but different pronunciation and/or meaning, like pencil "lead" vs. "lead" troops) are homographs in all the languages, as well.
- All the languages are written using the same character set.

As anyone who's ever learned another language can tell you, none of these are particularly realistic; some are laughably silly.   But the simplifications actually make the language interactions more puzzle-like, since you've got more information available to you than just the set of words that you know.

## What are we trying to build?

For a game I'm writing, I'm looking for a similar mechanism.   Like NMS, this game is extremely--almost pathologically-procedural, at least to the limits of what a single developer can produce in a reasonable time.  (I've resorted to asset store monsters, for example).

One planned element here is procedural pasts.  At world creation time, the last few thousand years of "history" are procedurally generated.  Each empire's ebb and flow are recorded, so that at any given time we can say things like "this particular area was held by the \'Empire of Bob\' 2000 years ago, so if you have ruins/dungeons/castles from that time, they're probably Bobian."   This stuff then feeds into the quest, lore, and location generators.  Similarly, the state of the generator at the "present time" determines political boundaries.

So I want each of these empires, kingdoms, etc.—both past and present—to have languages associated with them.   Often it'll be one-to-one: an empire speaks a specific language.  But it need not be:  sometimes there will be multiple "competing" languages within and empire (especially if it just took over lands that used to be owned by another), and often a language will be shared across empires (especially if friendly, or parts of the same historical empire.).  In the real world, languages are sometimes separated by political boundaries, but far more often by either geography, culture, or history.

In a nutshell, though, we don't know how many languages we're going to need, it can be arbitrarily many, or few, depending on what the generator produces.   And we'd like an additional requirement:  these languages shouldn't be just random strings of pronounceable characters.    They should be _visually distinct_ from each other.   Players should be able to distinguish languages from each other by appearance, at least most of the time.

So we want a system to generate arbitrarily many distinct languages, each of which can be "learned" piecemeal in the _No Man's Sky_ style.

## What makes up a language?

The dictionary tells us that a "language" is...well, it doesn't actually matter, because don't actually want a language.  We want something that _looks_ and _works_ just enough like a natural language to be a game substitute for one—similar to how we abstract "injury" into "hit points" or the uneven task of learning skills into "skill trees" of one sort or another.

Tolkien was a master of creating languages (he used real languages as a base).   Even ignoring the writing system glyphs, you could tell them apart:  elvish was full of L and N and vowel sounds, the dwarves used lots of K, H, D, and U.  Men (by which he mean "all humans"—_The Hobbit_ was published in 1937) tended toward old-English-y names with lots of E's and O's.

Star Trek's _Klingon_, meant to be the harsh tongue of a warrior race, leans heavily into hard consonants: Q, K, and P, and loves glottal stops (the hard break between consonant sounds that don't blend into each other):  "Q'pla" and the like; a sort of "coughing" language that would probably be impractical in real life.  Maybe the closest approximation being something like the San language or German.

But if we just make tables of letter frequencies and put them together, we tend to get unpronounceable messes:  "BHGLUM", "UEILNL", and the like.   That might actually work for Klingon, but in general, it's not going to give us what we want.    We could try to construct rules like "consonants and vowels need to alternate", which will produce always-pronounceable strings, but with more regularity than we'd like.

## Syllables to the rescue

Luckily, we've got a better construct to build from: the syllable.    Syllables are pronounceable sub-components of a word; effectively they're the "units" from which words are build.    Typically, each one represents a vowel sound and some optional surrounding consonants.

What really sets the sounds of languages apart is their choice of syllables.   English is a weird outlier:  because of "borrowing" from every language under the sun, it has something like 16,000 unique syllables in its lexicon.    Mandarin Chinese, on the other hand, has a mere 400 or so, although these are multiplied by four or five different "pronunciation frequencies" or "tones," to give about 1700 real ones.   

Even Mandarin is far more than we need.   Depending on who you ask, what you mean by "word" (are names included? Are plurals/tenses separate words?), and the specific language you're asking about, most people use about 10,000 different words in daily speech, and languages run somewhere between 80,000 and 150,000 total words.

But we don't care about the total words, much.   It would be a rare game whose text included all of  "verdant," "riboflavin," and "calligraphic."    We just need our language to be capable of covering any reasonable set of words that _we are actually going to use_.   I've seen estimates that a mere 1000 words is enough to read a newspaper, even if we double or triple that (say, a game like _Skyrim_ with hundreds of "books" in it to read), we're still not talking any significant fraction of a real language's scope.

So let's say we choose a target of 5,000 unique constructible "words" in our language.   How many syllables do we need?

If words are one syllable each, we need 5000 of them.    But if words are just two syllables, suddenly we need only about 70 of them $(\sqrt{5000})$.   At three syllables, it's ($\sqrt[3]{5000}$),  a mere 18 syllables.    Since most languages support words of varying length (in fact, average word length in syllables is one of the distinguishing characteristics of languages), or we allow homographs, we could get by with even fewer.

### Not all Syllables are equal

So far, we've got two "knobs" we can adjust in order to create our languages:

- Number of syllables per word
- The number of available syllables.

Next, we should consider the syllables themselves.  This is where we give a language it's character.  Consider, for example, a "soft" language (like Tolkien's Elvish) where we emphasize lilting or rolling consonants (like L, H, M, N) and vowel sounds:

```C#
private readonly static List<string> softSyllables = new List<string>() 
        {
          "la", "le", "lu", "li", "lo", "ly",
          "na", "ne", "nu", "ni", "no", "ny",
          "ha", "he", "hu", "hi", "ho", "hy",
          "ma", "me", "mu", "mi", "mo", "my",
          "al", "el", "ul", "il", "ol", "yl",
          "an", "em", "um", "im", "om", "ym",
          "an", "en", "un", "in", "on", "yn",
        };
```

This is C#, because the implementation is going to be in Unity, but the language doesn't matter much.   That's 35 syllables right there, and even using just that, we can get pleasant-sounding nonsense like ""noulmima na om olloully" or "emmuno lo nyhymu ynny."     The regularity of the 2-character syllables produces a sort of artificial-looking regularity to the generated text, but we can work with that.

A different set of syllables, heavy on sibilants (S, soft C) and hard consonants (K, P, T): 

```C#
private readonly static List<string> hardSyllables = new List<string>()
        {
          "ka", "ke", "ku", "ki", "ko", "ky",
          "sa", "se", "su", "si", "so", "sy",
          "ta", "te", "tu", "ti", "to", "ty",
          "pa", "pe", "pu", "pi", "po", "py",
          "ak", "ek", "uk", "ik", "ok", "yk",
          "at", "et", "ut", "it", "ot", "yt",
          "as", "es", "us", "is", "os", "ys",
        };
```

This gives us phrases like: "eskiakit ystuet ikas ytteak" and "taok ok ik ikpa."   Again, there's too much repetitiveness and pattern here, but again, that's because our syllables set is somewhat uniform and limited.

These are sort of the "extremes" of a syllabic spectrum.   If we define several sets, ranging from lilting to harsh, we then have a third knob for our languages:  what sets of syllables are available to it.   Something like English or German might distribute pretty easily across the whole spectrum.   Spanish-like languages would tend toward the softer end, something like Russian (or Klingon)  toward the harder end.  (Fantasy languages will likely go further out on the spectrum than natural languages will, because natural languages tend to evolve toward more easily-spoken forms over time.) 

So if we provide a way to generate a distribution (so many syllables from group A, so many from group B, etc.), we can use it to generate languages of different "character."

There are hundreds, maybe thousands, of potential syllables using roman/English alphabets, spellings, and pronounciation rules.  It's not necessary to provide an exhaustive list -- a few dozen at most should be enough to generate a rich set of languages.  (I've provided my lists in the code at the end of this article)

### A Møøse Once Bit My Sister

Since we're generating for a written form, and not too concerned about pronunciation, we can also use diacritical marks (those little accents, diaereses, umlauts, grave accents, circumflexes, macrons, tildes, etc.) more or less as decorations rather than to fulfill their usual functions of altering pronunciation, aspiration, or emphasis.   By building syllable sets that include diacritical forms, we can then choose (or randomly assign) languages to use them or not.

There are some caveats to doing this.   The first is that we're likely to generate things that range from nonsensical to outright offensive to speakers of the languages whose marks we're borrowing (e.g. there's no such thing as an "e-umlaut" in German, but Unicode lets us generate one easily enough: "ë".)

If the system you're using is _not_ Unicode-savvy, the risks become much higher.   A generation of computer users grew up with "smart quotes" from a Mac rendering as gibberish on a PC—and vice versa—even if you were using Microsoft Word on both; you still see these weird characters show up on the web occasionally.    And the encoding set of your language or OS may be the least of your problems:  any new glyph you use has to be present in whatever font you're using; which is a sort of hit-or-miss proposition when you're using characters "borrowed" from languages other than the ones the font is meant for.   We've all seen the "square" character rendered in place of some missing glyph (or lots of them) from a font.

Still, if you're confident in your encoding and font, diacriticals can add another visual distinction to a language, especially if you use different selections of them for different languages.

We could go even further down this rabbit hole and generate whole new glyphs (like _Ultima_'s rune writing) for our systems, but that just makes the font problem even worse.   You see this a lot in science fiction movies:  "alien" alphabets that are full of dots and symmetric geometric forms.

## OK, Now do we put this together?

We want to provide our developer with at least the following capabilities:

- Generate new synthetic languages
- Provide a string in "real" language and have that translated into the equivalent synthetic language string.   For convenience, we probably want to do this at the word level as well as at being able to provide a longer string of text.
- Provide a string in "real" language, along with a "dictionary" (in the literary, not programming, sense) of words that the player "knows", and have it translated into the equivalent synthetic language string with the "known" words indicated in the real language.

Note that translations are always real -> synthetic, not the other way around.   The app will deal in "real" strings, and convert them to synthetic (or part-synthetic) when presenting them to the users.

### Generating the Language Patterns

Our first step is to be able to choose values for all those "knobs" we discussed earlier:   syllable count ranges, which syllable sets to use, how many syllables to use from each set, whether we use punctuation (and if so, which punctuation) as syllable-separators, and the like.

We'd like to spare our caller needing to pick values for all those knobs, and ideally allow them to set none at all—if they wish—for a completely random one.

So we'll define a couple of "categories" here that categorize the language more simply:

```c#
 // Determines whether the language picks mostly from
    // the "soft," "middle," or "hard" syllables
    // RANDOM chooses one of these values with equal probability.
    public enum LanguageBias
    {
        RANDOM,
        SOFT,
        MIDDLE,
        HARD,
    }

    // Determines the number of syllables used for words (simple = fewer),
    // and the number of total syllables in the language (simple = fewer here, too). 
    // RANDOM generates a random value.
    public enum LanguageComplexity
    {
        RANDOM,
        SIMPLE,
        MEDIUM,
        COMPLEX,
    }
```

In both cases, we have a `.RANDOM` value that we can use if the developer doesn't want to think about it.

We can then define a bunch of properties for our language:

```c#
public class Language
{
    // These are the actual syllables that make up the language.   Something like
    // 20 is probably enough for most purposes (remember that the entire vocabulary
    // necessary will only have to cover the in-game strings that occur, so a few thousand
    // words will cover it.
    [JsonProperty] public List<string> syllables;

    // These should add up to 100.  They are the chances that a given word will
    // be 1, 2, 3, 4... syllables, respectively.   So {10, 80, 10} would mean that
    // most words in the language are 2 syllables, but 10% are 1 syllable, and 10% are 3.
    // If the language is very regular: (all words are the same number of syllables, or
    // there are only a couple of possibilities), you'll want more syllables in the list
    // above to cover the vocabulary.
    [JsonProperty] public int[] percentSyllableCountChance;

    // If true, the language uses punctuation to separate syllables within a word,
    // e.g. O'clock, brick-a-brack.
    // if true, "punctuations" holds the list of available characters, and
    // "punctuationChance" is the percent chance that any two syllables will
    // have a punctuation mark between them, and "leadPunctuationChance" is the
    // percent chance (which could be higher or lower than punctuationChance) that
    // there will be a punctuation mark specifically after the first syllable.
    // (Real, Fantasy, and Science fiction "languages" seem to emphasize that form a lot. 
    // Q'pla! T'Challa, O'Callahan).
    [JsonProperty] public bool usesPunctuationDelimeters;
    [JsonProperty] public string[]? punctuations;
    [JsonProperty] public float? punctuationChance;
    [JsonProperty] public float? leadPunctuationChance;

    // The actual "dictionary" of associations between English (or whatever) and
    // the synthetic language.
    [JsonProperty] public Dictionary<string, string> dictionary;

    // The actual name of the language, in it's own language.
    [JsonProperty] public string languageName;
```

Note that I'm using some C# nullables here, for the potentially-unnecessary punctuation dictionary.  Unity's Mono support doesn't allow nullables in older versions; so if your compiler complains, just get rid of the "?" characters in the definition of `punctuations`,` punctuationChance`, and `leadPunctuationChance.`

Let's write a constructor:

```c#
 public Language(LanguageBias bias = LanguageBias.RANDOM,
        LanguageComplexity complexity = LanguageComplexity.RANDOM)
    {
```

The developer who wants a particular harshness and complexity can choose it, but called without arguments, we'll just choose at random.    For harshness bias, that's pretty easy:

```C#
// Choose a bias emphasis.
LanguageBias useBias = bias;
if (bias == LanguageBias.RANDOM)
{
    switch (UnityEngine.Random.Range(0,3))
    {
        case 0: useBias = LanguageBias.SOFT; break;
        case 1: useBias = LanguageBias.MIDDLE; break;
        default: useBias = LanguageBias.HARD; break;
    }
}
```

This makes the odds even for each possibility, but you can fiddle with the values, if you like.  

> Side Note:  As you're about to notice, I'm using a lot of "magic numbers"—rather than constants.  This is an "author trick" to make numbers appear near their usage in context.  This is great for authoring blog articles, but sucks for maintenance, where you'd like to have them all in an easily-found block.   Once you're done fiddling and have dialed in values you like, you may want to consider changing them to proper constants.

"Complexity" is a little tougher, we're using it as a proxy for both the total number of syllables in the lexicon and the average number of syllables per word.

```C#
// And a number of syllables (a crude measure of language complexity).
int numSyllables = UnityEngine.Random.Range(40, 120);
percentSyllableCountChance = new int[] { 25, 30, 30, 10, 5 };

switch (complexity)
{
    case LanguageComplexity.SIMPLE:
        numSyllables = UnityEngine.Random.Range(40, 60);
        percentSyllableCountChance = new int[] { 50, 40, 10 };
        break;
    case LanguageComplexity.MEDIUM:
        numSyllables = UnityEngine.Random.Range(70, 105);
        percentSyllableCountChance = new int[] { 45, 35, 20 };
        break;
    case LanguageComplexity.COMPLEX:
        percentSyllableCountChance = new int[] { 15, 35, 25, 20, 5 };
        numSyllables = 120;
        break;
    default: break; // Already covered
}

syllables = ChooseSyllables(useBias, numSyllables);
```

Again, these are numbers that worked pretty well for me; you can change any of these numbers to match your own needs.   The main constraint here is the number of syllables available in the sets.   Mine are relatively small; if you have a _lot_ of potential syllables, the `numSyllables` value can go higher and you can bias more toward shorter words.   Real world languages tend to settle in around 1-3 syllables for most common words, since it literally makes speaking easier.  We're not considering word rarity here (in English, at least, significantly multisyllabic words tend to be used less, e.g. "significantly multisyllabic" vs. "long").

We'll come back to `ChooseSyllables`; it just picks a random set of syllables for the language, biased by `useBias`.

I want to use punctuation to add a little flavor to my languages, but I don't want to use it very often, or very much of it, to prevent the languages looking like caricatures.   So I'm fixing some values here.    It would make sense to have some rules (e.g. "harsh languages have more punctuation"), but I didn't bother for my needs.

```c#
// We'll use a fixed 15% chance that our language uses punctuation delimiters
usesPunctuationDelimeters = UnityEngine.Random.Range(0,100) < 15;

if (usesPunctuationDelimeters)
{
    // Set these to small, fixed values for now.  We can be more
    // selective later, if we need more variation.
    punctuations = new string[] { "-", "'", "*" };
    punctuationChance = 10f;
    leadPunctuationChance = 35f;
}
```

Again, if your compiler complains about C# nullables (or you just don't want to use them), get rid of the "if" and always set those values, even though they won't be used.  Note that the two "...Chance" variables are floats, not integers, to allow greater fine-tuning (like English, where hyphenated words are well under 1% of the language).

Finally, we just initialize the dictionary Dictionary (sigh...) that will hold our translations as we make them, and then generate a name for our language (in itself, of course).

```c#
    dictionary = new Dictionary<string, string>();
    languageName = "";
    GenerateLanguageName();
}
```

### Utilities

Let's write a couple utility functions.   We've seen one already, `ChooseSyllables()`, which selects the syllable set that will be part of our language.

```c#
public static List<string> ChooseSyllables(LanguageBias bias, int number)
{
    List<string> chosen = new List<string>();
    List<string> favorite, secondBest, thirdBest;

    // Order our lists by how much we like them.
    switch (bias)
    {
        case LanguageBias.SOFT:
            favorite = softSyllables;
            secondBest = middleSyllables;
            thirdBest = hardSyllables;
            break;
        case LanguageBias.HARD:
            favorite = hardSyllables;
            secondBest = middleSyllables;
            thirdBest = softSyllables;
            break;
        case LanguageBias.MIDDLE:
        default:
            favorite = middleSyllables;
            secondBest = softSyllables;
            thirdBest = hardSyllables;
            break;
    }

    while (chosen.Count < number)
    {
        int percentileRoll = UnityEngine.Random.Range(0, 100);
        string syllable;

        if (percentileRoll < 70)
        {
            syllable = favorite[UnityEngine.Random.Range(0, favorite.Count)];
        }
        else if (percentileRoll < 95)
        {
            syllable = secondBest[UnityEngine.Random.Range(0, secondBest.Count)];
        }
        else
        {
            syllable = thirdBest[UnityEngine.Random.Range(0, thirdBest.Count)];
        }

        if (!chosen.Contains(syllable))
            chosen.Add(syllable);
    }

    return chosen;
}
```

This isn't going to win any efficiency awards.  That `while` loop does an awful lot of `List.Contains()` calls (which, since our list is unsorted, is O(n) each time), and after all that effort will throw away the syllable it just picked if it happens to already be in the list.

The actual performance hit here is inversely proportional to the difference between the total number of syllables available in our lists and the number we're looking for.    If we've got lots more syllables available than we need, it's not bad.   As the "excess" becomes smaller, the odds of a collision get higher, and we end up discarding more syllables.   If the excess becomes very small, this could take a very long time to "find" those last few free syllables.

If the excess is negative (we're asking for more unique syllables than exist), this will loop forever, and you'll get a crash course (well, a "hang course") in how to recover from a hang.  (Hint -- kill the Unity Editor by whatever the standard means is on your platform).

So, let's add a check at the beginning:

```C#
int syllablesAvailable = softSyllables.Count + middleSyllables.Count + hardSyllables.Count;
Debug.Assert(syllablesAvailable > number * 1.5f,
    $"ASSERT: We're looking for {number} syllables out of {syllablesAvailable}, which will likely take a long time.");

```

This will check (in debug builds) that we've got at least 150% of the number of syllables that we're asking for.    That doesn't actually _fix_ the problem, but it should make us aware of it.

An easier case is `SyllableCount()`, which just used that `percentSyllableCountChance` array to choose the a number of syllables according to the provided probability distribution:

```C#
private int SyllableCount()
{
    int num = UnityEngine.Random.Range(0, 100);
    int outNum = 0;
    while (num > percentSyllableCountChance[outNum])
    {
        num -= percentSyllableCountChance[outNum];
        outNum++;
    }

    return outNum + 1;
}
```

Once again, the Software Engineer in me is squinting unpleasantly at this.   The problem is that "100."   If we have no bugs, and our "percent chances" are actually percents (i.e. add up to 100), then this works fine.   But if we happen to give it an array that sums to more than 100, then the higher values will never get picked.    And if we give it an array that sums to less than 100, this will crash if the random generator picks a number greater than our maximum!

So are you feeling lucky?   I'm not.   Let's fix it.

```C#
private int SyllableCount()
{
    int chanceSum = 0;
    foreach (int chance in percentSyllableCountChance)
    {
        chanceSum += chance;
    }
  
    int num = UnityEngine.Random.Range(0, chanceSum);
    int outNum = 0;
    while (num > percentSyllableCountChance[outNum])
    {
        num -= percentSyllableCountChance[outNum];
        outNum++;
    }

    return outNum + 1;
}
```

That works better—although it the chance array were empty, or if it held negative values, we could get weird effects.    We should return a reasonable default in the first case, and assert on the latter:

```C#
private int SyllableCount()
{
    if (percentSyllableCountChance.Length == 0) { return 2; }

    int chanceSum = 0;
    foreach (int chance in percentSyllableCountChance)
    {
        chanceSum += chance;
        Debug.Assert(chance > 0, 
           "ASSERT: Zero or negative probability in percentSyllableCountChance.");
    }

    int num = UnityEngine.Random.Range(0, chanceSum);
    int outNum = 0;
    while (num > percentSyllableCountChance[outNum])
    {
        num -= percentSyllableCountChance[outNum];
        outNum++;
    }

    return outNum + 1;
}
```

Some of you may note that that summing `foreach` loop is the "Reduce" of the "Map / Reduce" algorithm pair, available in effectively every code framework in the world....except .NET.   Linq gives you `Select` and `Aggregate`, which are rough equivalents, albeit with a syntax that makes kittens cry.   I'll stick with the readable `foreach` loop.  You're welcome.

## Generating Words

We're ready for our big debut.   Let's actually build words in our new language!  It's sort of anti-climactic, after all that setup:

```C#
public string GenerateWordFor(string baseLanguageWord)
{
    if (!dictionary.ContainsKey(baseLanguageWord))
    {
        int numSyll = SyllableCount();
        string word = "";
        for (int i = 0; i < numSyll; i++)
        {
            word += syllables[UnityEngine.Random.Range(0, syllables.Count)];
            if (usesPunctuationDelimeters && i < (numSyll - 1))
            {
                float percent = UnityEngine.Random.Range(0f, 100f);
                if (percent < ((i ==0) ? leadPunctuationChance : punctuationChance))
                {
                    int choice = UnityEngine.Random.Range(0, punctuations.Length);
                    word += punctuations[choice];
                }
            }
        }

        dictionary[baseLanguageWord] = word;
    }

    return dictionary[baseLanguageWord];
}
```

We start by making sure we haven't already seen this word.  If we have, it's in our dictionary (literary sense, although it's also a Dictionary in the software sense), and we just return the previous translation.

Otherwise, it's fairly straightforward.   Pick a number of syllables, add one at random, and slap a little punctuation on there, if our language and random numbers agree.  (Note that we never put punctuation _after_ the last syllable, which is just a stylistic choice.)

### Generating Longer Text

So now we've got all the pieces we need to do longer text:  we just divide it into words, pass each word through `GenerateWordFor()` and concatenate them all back together.    But that quick algorithm belies some complicated decisions.   Specifically, what's a word?

```C#
 public string GenerateTextFor(string baseLanguageText)
    {
        string[] words = baseLanguageText.Split(' ');
        string outString = "";
        foreach (string word in words)
        {
            outString += GenerateWordFor(word) + " ";
        }

        // Get rid of that last space.
        outString = outString.TrimEnd();

        return outString;
    }
```

That's our baseline case.   Maybe it's good enough for some uses, but it's pretty primitive.   We're just treating every run of characters that's not a space as a word.   "Don't" is a word.  But so is "Hello.", _including the period._  Or "funny-looking".   

From a quick test run:

```text
"Hello, my fine countryman." in language: aklon is "ni dap foeza vo"
```

Just at a first glance, we've lost the comma, the period at the end, and the capitalization of the first letter.   "Hello," "Hello" "hello" "hello." would all be treated as different words.   If our "sources" language is something like Chinese, that doesn't tend to use spaces between words, it's even less useful.

We'll gloss over that last point, except to note that we may need to make different versions of GenerateTextFor() for some different (real) input languages.  

But for our "generic Roman language" case:

- We'd like words to be translated case-insensitively ("hello" and "HELLO") as the same word.
- We'd like to maintain first--character--case.   If a word is capitalized in the source text, it should be capitalized in the translation, too.    But we'll live with words in all-caps or other partial-capitalization patterns being considered as all-lower-case.    But even in this case, "Hello" and "hello" should translated to the same word (say, "Blah" and "blah"). 
- Many people don't differentiate between hyphens and dashes when writing, so it's probably safest to just treat hyphens as word breaks, unless we know that our text uses them only as hyphens.   This will result in hyphenated words remaining hyphenated after translation, which is weird, but not as weird as having things like dashes disappear.
- We probably want to treat a single-quote as part of a word (don't, doesn't, can't, o'clock).
- But most other punctuation should be considered a separator, and put back into the string after translation in the same locations.

So `string.Split(' ')` isn't going to do it any more.

What will?   If you're implementing in a language other than C#/.NET, you may want to just stop reading now and go off on your own.   Many modern languages/frameworks have fairly sophisticated parsing capabilities, and you might be able to get some of this stuff for free.

But we'll go with a simpler algorithm.   We'll keep a list of punctuation we consider part of a word (just single quotes for now, although you could add hyphen if you're confident in your source texts), and letters.   We'll run through the text character by character:

- If it's "part of a word", we'll just append it to the "current" word we're making.
- If not, we'll translate the "current word" (if any), put the translation in the output string, clear the "current word," and then just echo the character we found to the output string, too.

That's not too bad to implement in any language.

First, a utility routine:

```C#
public string TranslationForWordWithCase(string baseLanguageWord)
{
    if (baseLanguageWord.Length == 0) { return baseLanguageWord; }

    string outWord = GenerateWordFor(baseLanguageWord.ToLower());
    // If the first character is uppercase, but the second one isn't, convert
    // the first character of the output word to upper case.
    if (char.IsUpper(baseLanguageWord, 0) &&
        !(baseLanguageWord.Length > 1 && char.IsUpper(baseLanguageWord, 1)))
    {
        outWord = char.ToUpper(outWord[0]) + outWord.Substring(1);
    }

    return outWord;
}
```

This just handles our capitalization rules.  A word whose first character is capitalized will produce a translation whose first character is capitalized.    But something like "HELLO" will be treated as all lower case.    As usual, you can fiddle with the code if you want different rules.

With that, we can write our long text handler.

```C#
public string GenerateTextFor(string baseLanguageText)
  {
      string currentWord = "";
      string outString = "";

      foreach (char c in baseLanguageText)
      {
          if (char.IsLetter(c) || c == '\'')
          {
              currentWord += c;
          }
          else
          {
              if (currentWord.Length > 0)
              {
                  outString += TranslationForWordWithCase(currentWord);
                  currentWord = "";
              }
              outString += c;
          }
      }

      // Repeat this check, just in case we didn't end with punctuation.
      if (currentWord.Length > 0)
      {
          outString += TranslationForWordWithCase(currentWord);
          currentWord = "";
      }

      return outString;
  }
```

The use of `char.IsLetter()` means that numbers/digits are considered punctuation.   If you don't want that, you can either use `IsLetterOrDigit()` or just write out your numbers rather than using digits.

### Partial translations

The "translation" mechanism in _No Man's Sky_ works by printing out the synthetic text, but with "known" words replaced by their real language equivalent:  "Blah blahh, bblah cucumber bbllaahh mauve," or suchlike.   

There are two mechanism for "knowing" a word.   The player can actually learn them by various means ("knowledge stones" on planet surfaces, speaking with aliens, alien monuments, etc.).   Those are permanently learned and will always translate once known.     The second mechanism are "machine translators," which translate a small number of "additional" words on a translation-by-translation basis.   These words are translated for the current interaction, but become unknown again afterward.

I don't like the exact implementation of that second mechanism—and in fact I'm not sure if my game will even have anything like it—but the idea is sound.    So I'm going to modify it slightly; I'm thinking something like a "linguistics" skill that gives the player a small chance of "guessing" unknown words based on "context."   

The last routine we're going to generate here is one that generates synthetic language texts with the base language filled in for known words. 

This will take four arguments:   

1. The base language text to translate.
2. An array of "known" words in the base language
3. A percent chance to "guess" each other word
4. A boolean indicating whether to surround translated words with brackets.

That last gives us the option for something like "Blah blahh, bblah [cucumber] bbllaahh [mauve]," which makes the "known words" more obvious.  (It'll use hard brackets for known words and angle-brackets for guessed ones.)

This function is going to be very similar to `GenerateTextFor`.  In fact, it's going to be _so_ similar, that we probably don't want to write a separate function for it at all, but rather just pass in some optional arguments to `GenerateTextFor`.

```C#
public string GenerateTextFor(string baseLanguageText,
                  List<string> knownWords = null,
                  float percentGuess = 0f,
                  bool useBracketsForKnown = true)
{
    string currentWord = "";
    string outString = "";

    foreach (char c in baseLanguageText)
    {
        if (char.IsLetter(c) || c == '\'')
        {
            currentWord += c;
        }
        else
        {
            if (currentWord.Length > 0)
            {
                if (knownWords != null && knownWords.Contains(currentWord.ToLower()))
                {
                    if (useBracketsForKnown)
                        outString += $"[{currentWord}]";
                    else
                        outString += currentWord;
                }
                else if (UnityEngine.Random.Range(0f, 100f) < percentGuess)
                {
                    if (useBracketsForKnown)
                        outString += $"<{currentWord}>";
                    else
                        outString += currentWord;
                }
                else
                {
                    outString += TranslationForWordWithCase(currentWord);
                }
                currentWord = "";
            }

            outString += c;
        }
    }

    // Repeat this check, just in case we didn't end with punctuation.
    if (currentWord.Length > 0)
    {
        outString += TranslationForWordWithCase(currentWord);
        currentWord = "";
    }

    return outString;
}
```

There's a little repetition in there that we could factor out, but it's not bad.

## Testing

So, does it work?

I come from the mobile development world, where unit testing is relatively rare.   And I'm not a fan of unit testing for most things in the first place—in my experience, test-driven development in general never pays for itself in higher quality; just higher effort.   A lot of man hours are spent trying to make things unit-testable that would be better spent making them self-diagnostic.  Programming cultures that attempt to institutionalize full-automated-testing for code invariably produce thousands of tests that never fail—typically because they're of the "Does 2 plus 2 still equal 4 today?" variety (or worse, tests that _do_fail intermittently, with "automatic re-running" turned on for as many times as necessary for it to pass), but that never seems to map to higher quality, fewer bugs, or easier integrations.   It does check a box on a report, though.

Me?  I'll take a skilled human tester, any day.   Unlike my experiences with automated tests, human QA's have identified that "weird quirk" for me more times than I can count.

All of that having been said, if a system is:

- **Stateless**, in the sense that it doesn't require the system to be in a given state before it will work.  (For example, in many apps, most functions don't work unless the user is "logged in," and the output of those functions will vary based on what the user has done recently and in what order.)
- **Predictable**, in the sense that the outputs of the system are either non-random or the randomness can be seeded for repeatability,
- **Independent**, in the sense that it does not depend on fallible external capabilities like networking, and
- **Measurable**, in a broad sense

Then unit testing can provide both a sort of automated "checklist" for the developer, and a place for other developers to look for examples of the system being called.

Our language generation fails the "predictable" condition, since it generates both the language construction and the "guessing" substitutions randomly.   But maybe we can work around that.

### Non-Random Properties

We want our system to have these properties:

- Given the same input string, a Language object should produce the same output string on every call.
- Different Language objects should produce different outputs for the same output string.
- Language objects should produce different outputs for different input strings.
- If we pass in a "known" array that contains a word in the input string, the output string should contain that word verbatim.
  - And if we've got "useBracketsForKnown" on, that would should have [brackets] around it.
- All of the above assume a "percentGuess" of 0.    If we pass "percentGuess" of 100, the output string should contain all the words of the input string:
  - if "useBracketsForKnown" is on, every word not in the known list should have <angle brackets> around it.
  - If "useBracketsForKnown" is off (false), the input string should be exactly the same as the output string. 

All those can be tested independently of randomness.

Setting up Unity's unit testing (a variant of NUnit) is a topic for another time.  But assuming you've built one, we can build a test easy enough:

```C#
// A Test behaves as an ordinary method
[Test]
public void LanguageSimplePasses()
{
    // Use the Assert class to test conditions
    string firstTest = "Hello, my fine countryman!";
    string secondTest = "Hello, you awful countryman.";

    List<string> known = new List<string>();
    known.Add("countryman");
    known.Add("you");

    Language foreignSpeak = new Language(Language.LanguageBias.SOFT, Language.LanguageComplexity.SIMPLE);

    Debug.Log(foreignSpeak.GenerateTextFor(firstTest));
    Debug.Log(foreignSpeak.GenerateTextFor(secondTest));
    Assert.AreEqual(foreignSpeak.GenerateTextFor(firstTest), foreignSpeak.GenerateTextFor(firstTest));
    Assert.AreNotEqual(foreignSpeak.GenerateTextFor(firstTest), foreignSpeak.GenerateTextFor(secondTest));

    Debug.Log(foreignSpeak.GenerateTextFor(firstTest, known, 0, true));
    Debug.Log(foreignSpeak.GenerateTextFor(secondTest, known, 0, true));
    Debug.Log(foreignSpeak.GenerateTextFor(secondTest, known, 100, true));
    Debug.Log(foreignSpeak.GenerateTextFor(secondTest, known, 100, false));

    Assert.IsTrue(foreignSpeak.GenerateTextFor(firstTest, known, 0, true).Contains("[countryman]"));
    Assert.IsTrue(foreignSpeak.GenerateTextFor(secondTest, known, 0, true).Contains("[you]"));
    Assert.IsTrue(foreignSpeak.GenerateTextFor(secondTest, known, 100, true).Contains("<awful>"));
    Assert.IsTrue(foreignSpeak.GenerateTextFor(secondTest, known, 100, false).Contains(secondTest));
}
```

The "Debug.Log()" lines there are to make the Unity Test Runner window echo results, so that we can see our languages in process.

The test passes, as we can see in the lower part of the Test Runner window.

```text
LanguageSimplePasses (0.017s)
---
Im, ad nel-yw oswo!
Im, va ceth oswo.
Im, ad nel-yw [countryman]!
Im, [you] ceth [countryman].
<Hello>, [you] <awful> [countryman].
Hello, you awful countryman.
```

### Non-Predictable Properties

How about the rest?

- Languages with different settings should "look different."
- A few languages should have some punctuation in words.
- Guess amounts 0 < x < 100 should produce roughly a corresponding percent of <guessed> words.
- Two different input strings with words in common should results in the same "translation" for those words.
- Leading capitals and punctuation except for single-quotes should be in the same relative places in the output string as it was in the input string.

The first three points are going to depend on our random number generator.  The last two don't, but they're both something that's trivial for a human to check, but relatively annoying to write code for.

We _could_ make the random stuff non-random by using `UnityEngine.Random.seed=42` or some other fixed number before doing them.   But we're not going to, because doing so creates a subtle dependency:  we'll only get the same outputs—even with the same seed—if the number and order of calls to the random number generator don't change.    Any tests we build based on the results of a single seed are going to be fragile:  Any change (successful or not, valid or not, minor or not) to how the algorithms use `Random` will make all our tests fail.  And then someone's going to have to spend a day figuring out WHY the tests failed, even if the code is still working perfectly.    That sort of nonsense is why I'm not a Unit Test believer.

But we still need the ability to verify that the code is working as intended, and honestly, we'd be better to rely on human judgement for that.

So we'll just add a bunch of output to the end of the existing test, which will produce a "Manual Evaluation Output" section that we can just look over:

```C#
// This isn't checked as part of the test, it just generates more output 
        Debug.Log("----");
        Debug.Log("Manual Evaluation Output");
        foreignSpeak = new Language(Language.LanguageBias.SOFT, Language.LanguageComplexity.SIMPLE);
        Debug.Log("Simple, Soft Language");
        Debug.Log($"\"{firstTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(firstTest)}\"");
        Debug.Log($"Pass 2: \"{firstTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(firstTest)}\"");
        Debug.Log($"\"{secondTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(secondTest)}\"");
        Debug.Log($"Pass 2: \"{secondTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(secondTest)}\"");

        foreignSpeak = new Language(Language.LanguageBias.HARD, Language.LanguageComplexity.COMPLEX);
        Debug.Log("Hard, Complex Language");
        Debug.Log($"\"{firstTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(firstTest)}\"");
        Debug.Log($"\"{secondTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(secondTest)}\"");

        foreignSpeak = new Language(Language.LanguageBias.MIDDLE, Language.LanguageComplexity.MEDIUM);
        Debug.Log("Medium, Medium Language");
        Debug.Log($"\"{firstTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(firstTest)}\"");
        Debug.Log($"\"{secondTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(secondTest)}\"");

        for (int i = 0; i < 20; i++)
        {
            foreignSpeak = new Language();
            Debug.Log("Random Language");
            Debug.Log($"\"{firstTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(firstTest)}\"");
            Debug.Log($"\"{secondTest}\" in language: {foreignSpeak.languageName} is \"{foreignSpeak.GenerateTextFor(secondTest)}\"");
        }
```

Some sample output:

```text
LanguageSimplePasses (0.036s)
---
Ud, evud ussaln nolav!
Ud, omien ull nolav.
Ud, evud ussaln [countryman]!
Ud, [you] ull [countryman].
<Hello>, [you] <awful> [countryman].
Hello, you awful countryman.
----
Manual Evaluation Output
Simple, Soft Language
"Hello, my fine countryman!" in language: Osieh is "Woyn, ew uw in!"
Pass 2: "Hello, my fine countryman!" in language: Osieh is "Woyn, ew uw in!"
"Hello, you awful countryman." in language: Osieh is "Woyn, fiean alal in."
Pass 2: "Hello, you awful countryman." in language: Osieh is "Woyn, fiean alal in."
Hard, Complex Language
"Hello, my fine countryman!" in language: Es is "Phapi, qui ustabota thuixhagthu!"
"Hello, you awful countryman." in language: Es is "Phapi, lanet etyvxi thuixhagthu."
Medium, Medium Language
"Hello, my fine countryman!" in language: Uph is "Dar, heiph elong ungyth!"
"Hello, you awful countryman." in language: Uph is "Dar, asith harhat ungyth."
Random Language
"Hello, my fine countryman!" in language: Bihalsse is "Ubdakfieni, ni phalanthoephaph phalennu!"
"Hello, you awful countryman." in language: Bihalsse is "Ubdakfieni, bimy myhapkhu phalennu."
Random Language
"Hello, my fine countryman!" in language: Ullfaoballes is "Seoth, zo phi faob!"
"Hello, you awful countryman." in language: Ullfaoballes is "Seoth, eln foeongiswo faob."
Random Language
"Hello, my fine countryman!" in language: Cathfi is "Ieh, harbe has thi!"
"Hello, you awful countryman." in language: Cathfi is "Ieh, en dakubthe thi."
Random Language
"Hello, my fine countryman!" in language: Kyessiss is "Tyizz, quo teoxuk onso!"
"Hello, you awful countryman." in language: Kyessiss is "Tyizz, asax zuosquoosot onso."
Random Language
"Hello, my fine countryman!" in language: Athimthoha is "Olleb, lues elell issse!"
"Hello, you awful countryman." in language: Athimthoha is "Olleb, en panul issse."
Random Language
"Hello, my fine countryman!" in language: Eweih is "Syiehmidathing, ylnoln phiewill ylnnelan!"
"Hello, you awful countryman." in language: Eweih is "Syiehmidathing, lynnil kithuvme ylnnelan."
Random Language
"Hello, my fine countryman!" in language: Me is "Ellboba, yss ssadathssuoss bithuthuong!"
"Hello, you awful countryman." in language: Me is "Ellboba, ulfietu cethollkha bithuthuong."
Random Language
"Hello, my fine countryman!" in language: Thuunloavkith is "Thissiien, ssaki lonul ingoss!"
"Hello, you awful countryman." in language: Thuunloavkith is "Thissiien, khoki sy ingoss."
Random Language
"Hello, my fine countryman!" in language: Uphtuxa is "Kha, edosatazzcuth de q!"
"Hello, you awful countryman." in language: Uphtuxa is "Kha, ot sayk q."
Random Language
"Hello, my fine countryman!" in language: Ezzuw is "Tocath, fa ba hah!"
"Hello, you awful countryman." in language: Ezzuw is "Tocath, ethma kithdaphak hah."
Random Language
"Hello, my fine countryman!" in language: Ud is "Zuzu, phithe nohath okit!"
"Hello, you awful countryman." in language: Ud is "Zuzu, ixpaque us okit."
Random Language
"Hello, my fine countryman!" in language: Di is "Bo, oskyizz zoizz osxafaik!"
"Hello, you awful countryman." in language: Di is "Bo, awhahozz doezz osxafaik."
Random Language
"Hello, my fine countryman!" in language: Iph is "Queediss, ykulnqui ingphize q!"
"Hello, you awful countryman." in language: Iph is "Queediss, depu piadabhan q."
Random Language
"Hello, my fine countryman!" in language: Quaaddah is "Ke, xatenel qu fiphe!"
"Hello, you awful countryman." in language: Quaaddah is "Ke, etoxkho nel fiphe."
Random Language
"Hello, my fine countryman!" in language: Bedagix is "Sysu, vikhodar lanfephu khiathothse!"
"Hello, you awful countryman." in language: Bedagix is "Sysu, khu fa khiathothse."
Random Language
"Hello, my fine countryman!" in language: Ethssyha is "Eindathbe, dahse ssithyle ssiwuwa!"
"Hello, you awful countryman." in language: Ethssyha is "Eindathbe, issceth uth ssiwuwa."
Random Language
"Hello, my fine countryman!" in language: Quiud is "Ux*ssizuusdi, athath*kho qua*ythzi be!"
"Hello, you awful countryman." in language: Quiud is "Ux*ssizuusdi, se-ukit ti*xu be."
Random Language
"Hello, my fine countryman!" in language: Phaus is "Feekhase, thinol ky haiehobhagieh!"
"Hello, you awful countryman." in language: Phaus is "Feekhase, kha wypho haiehobhagieh."
Random Language
"Hello, my fine countryman!" in language: Hap is "Kha, lineldatdar akomwaphi sussawy!"
"Hello, you awful countryman." in language: Hap is "Kha, pho essaln sussawy."
Random Language
"Hello, my fine countryman!" in language: Quut is "Ssoaxqu, zait quike tias!"
"Hello, you awful countryman." in language: Quut is "Ssoaxqu, kathoqu izz tias."
```

That looks good to me.

# The Code

Here's the code, all in one place:

```C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Newtonsoft.Json;

// Implementation of a written language.
//
// This uses a No Man's Sky-like linguistic system.   Languages
// all have the same grammar, word order, etc. as the player's chosen
// language, but different actual words.
//
// Messages in game are parsed for words, and each word given a substitute
// in the "foreign" language.   This dictionary is maintained and extended
// as new messages are added, and sorted by the frequency of the real-world
// words seen so far.
//
// Players can, through various means, "learn" words of a language; when displayed,
// those words will be shown in their chosen language (and displayed differently)
// rather than the synthetic one.   Certain spells and effects may also
// allow the player to read the synthetic language directly.
//
// For the moment, we'll ignore homographs (words that are spelled alike, but are
// actually different words, the "lead" in "pencil lead" vs. "lead the troops.")
// The synthetic word that matches "lead" will be used for both.   Given the
// large numbers of other ridiculous assumptions being made here (words are 1:1 substitutes?),
// I doubt players will care about that one, much.

/// <summary>
/// This is a mechanismg for making multiple synthetic languages that are visually
/// distinct.  It basically selects subsets from one of several "syllable sets",
/// as well as probabilities for short vs. long words, frequency of repeated
/// syllables, and punctuation delimiters.   It will then use those elements to
/// create an infinite number of words, on demand.
/// 
/// ??? Someday might be nice to have "alphabetic-like" glyph sets, too, like Ultima's rune
/// languages.
/// </summary>
[Serializable]
[JsonObject(MemberSerialization.OptIn)]
public class Language
{
    // These are the actual syllables that make up the language.   Something like
    // 20 is probably enough for most purposes (remember that the entire vocabulary
    // necessary will only have to cover the in-game strings that occur, so a few thousand
    // words will cover it.
    [JsonProperty] public List<string> syllables;

    // These should add up to 100.  They are the chances that a given word will
    // be 1, 2, 3, 4... syllables, respectively.   So {10, 80, 10} would mean that
    // most words in the language are 2 syllables, but 10% are 1 syllable, and 10% are 3.
    // If the language is very regular: (all words are the same number of syllables, or
    // there are only a couple of possibilities), you'll want more syllables in the list
    // above to cover the vocabulary.
    [JsonProperty] public int[] percentSyllableCountChance;

    // If true, the language uses punctuation to separate syllables within a word,
    // e.g. O'clock, brick-a-brack.
    // if true, "punctuations" holds the list of available characters, and
    // "punctuationChance" is the percent chance that any two syllables will
    // have a punctuation mark between them, and "leadPunctuationChance" is the
    // percent chance (which could be higher or lower than punctuationChance) that
    // there will be a punctuation mark specifically after the first syllable.
    // (Real, Fantasy, and Science fiction "languages" seem to emphasize that form a lot. Q'pla! T'Challa, O'Callahan).
    [JsonProperty] public bool usesPunctuationDelimeters;
    [JsonProperty] public string[]? punctuations;
    [JsonProperty] public float? punctuationChance;
    [JsonProperty] public float? leadPunctuationChance;

    // The actual "dictionary" of associations between English (or whatever) and
    // the synthetic language.
    [JsonProperty] public Dictionary<string, string> dictionary;

    // The actual name of the language, in it's own language.
    [JsonProperty] public string languageName;

    private readonly static List<string> softSyllables = new List<string>()
        {
          "la", "le", "lu", "li", "lo", "ly",
          "na", "ne", "nu", "ni", "no", "ny",
          "ha", "he", "hu", "hi", "ho", "hy",
          "ma", "me", "mu", "mi", "mo", "my",
          "al", "el", "ul", "il", "ol", "yl",
          "an", "em", "um", "im", "om", "ym",
          "an", "en", "un", "in", "on", "yn",
          "lan", "len", "lun", "lin", "lon", "lyn",
          "nal", "nel", "nul", "nil", "nol", "nyl",
          "all", "ell", "ull", "ill", "oll", "yll",
          "aln", "eln", "uln", "iln", "oln", "yln",
          "eil", "ein", "eim", "eih", "iel", "ien", "iem", "ieh",
          "av", "ev", "uv", "iv", "ov", "yv",
          "va", "ve", "vu", "vi", "vo", "vy",
          "aw", "ew", "uw", "iw", "ow", "yw",
          "wa", "we", "wu", "wi", "wo", "wy",
        };

    private readonly static List<string> middleSyllables = new List<string>()
        {
          "ang", "eng", "ing", "ung", "ong",
          "fa", "fe", "fi", "fo", "fu",
          "fae", "fie", "fee", "foe", "fum",
          "dan", "dat", "dak", "dah", "dal",
          "dath", "dap", "dag", "dar", "das",
          "han", "hat", "hak", "hah", "hal",
          "hath", "hap", "hag", "har", "has",
          "kha", "khe", "khi", "kho", "khu", "khy",
          "tha", "the", "thi", "tho", "thu", "thy",
          "ath", "eth", "ith", "oth", "uth", "yth",
          "sa", "ssa", "se", "sse", "si", "ssi", "so", "sso", "su", "ssu", "sy", "ssy",
          "as", "ass", "es", "ess", "is", "iss", "os", "oss", "us", "uss", "ys", "yss",
          "ba", "be", "bi", "bo", "bu",
          "ab", "eb", "ib", "ob", "ub",
          "pha", "phe",  "phi", "pho", "phu",
          "aph", "eph", "iph", "oph", "uph",
          "cath", "ceth", "kith", "coth", "cuth", "cyth",
    };

    private readonly static List<string> hardSyllables = new List<string>()
        {
          "ka", "ke", "ku", "ki", "ko", "ky",
          "sa", "se", "su", "si", "so", "sy",
          "ta", "te", "tu", "ti", "to", "ty",
          "pa", "pe", "pu", "pi", "po", "py",
          "ak", "ek", "uk", "ik", "ok", "yk",
          "at", "et", "ut", "it", "ot", "yt",
          "as", "es", "us", "is", "os", "ys",
          "xa", "xe", "xi", "xu", "xo",
          "ax", "ex", "ix", "ux", "ox",
          "da", "de", "di", "du", "do",
          "ad", "ed", "id", "ud", "od",
          "q", "qua", "que", "qui", "quo", "qu",
          "za", "ze", "zi", "zo", "zu",
          "azz", "ezz", "izz", "ozz", "uzz"
        };


    // Determines whether the language picks mostly from
    // the "soft," "middle," or "hard" syllables
    // RANDOM chooses one of these values with equal probability.
    public enum LanguageBias
    {
        RANDOM,
        SOFT,
        MIDDLE,
        HARD,
    }

    // Determines the number of syllables used for words (simple = fewer),
    // and the number of total syllables in the language (simple = fewer here, too). 
    // RANDOM generates a random value.
    public enum LanguageComplexity
    {
        RANDOM,
        SIMPLE,
        MEDIUM,
        COMPLEX,
    }

    public static List<string> ChooseSyllables(LanguageBias bias, int number)
    {
        int syllablesAvailable = softSyllables.Count + middleSyllables.Count + hardSyllables.Count;
        Debug.Assert(syllablesAvailable > number * 1.5f,
            $"ASSERT: We're looking for {number} syllables out of {syllablesAvailable}, which will likely take a long time.");
        List<string> chosen = new List<string>();
        List<string> favorite, secondBest, thirdBest;

        // Order our lists by how much we like them.
        switch (bias)
        {
            case LanguageBias.SOFT:
                favorite = softSyllables;
                secondBest = middleSyllables;
                thirdBest = hardSyllables;
                break;
            case LanguageBias.HARD:
                favorite = hardSyllables;
                secondBest = middleSyllables;
                thirdBest = softSyllables;
                break;
            case LanguageBias.MIDDLE:
            default:
                favorite = middleSyllables;
                secondBest = softSyllables;
                thirdBest = hardSyllables;
                break;
        }

        while (chosen.Count < number)
        {
            int percentileRoll = UnityEngine.Random.Range(0, 100);
            string syllable;

            if (percentileRoll < 70)
            {
                syllable = favorite[UnityEngine.Random.Range(0, favorite.Count)];
            }
            else if (percentileRoll < 95)
            {
                syllable = secondBest[UnityEngine.Random.Range(0, secondBest.Count)];
            }
            else
            {
                syllable = thirdBest[UnityEngine.Random.Range(0, thirdBest.Count)];
            }

            if (!chosen.Contains(syllable))
                chosen.Add(syllable);
        }

        return chosen;
    }


    /// <summary>
    /// Constructor to generate one of these.
    /// </summary>
    public Language(LanguageBias bias = LanguageBias.RANDOM,
        LanguageComplexity complexity = LanguageComplexity.RANDOM)
    {
        // Choose a bias emphasis.
        LanguageBias useBias = bias;
        if (bias == LanguageBias.RANDOM)
        {
            switch (UnityEngine.Random.Range(0, 3))
            {
                case 0: useBias = LanguageBias.SOFT; break;
                case 1: useBias = LanguageBias.MIDDLE; break;
                default: useBias = LanguageBias.HARD; break;
            }
        }

        // And a number of syllables (a crude measure of language complexity).
        int numSyllables = UnityEngine.Random.Range(40, 120);
        percentSyllableCountChance = new int[] { 25, 30, 30, 10, 5 };

        switch (complexity)
        {
            case LanguageComplexity.SIMPLE:
                numSyllables = UnityEngine.Random.Range(40, 60);
                percentSyllableCountChance = new int[] { 50, 40, 10 };
                break;
            case LanguageComplexity.MEDIUM:
                numSyllables = UnityEngine.Random.Range(70, 105);
                percentSyllableCountChance = new int[] { 45, 35, 20 };
                break;
            case LanguageComplexity.COMPLEX:
                percentSyllableCountChance = new int[] { 15, 35, 25, 20, 5 };
                numSyllables = 120;
                break;
            default: break; // Already covered
        }

        syllables = ChooseSyllables(useBias, numSyllables);

        // We'll use a fixed 15% chance that our language uses punctuation delimiters
        usesPunctuationDelimeters = UnityEngine.Random.Range(0, 100) < 15;

        if (usesPunctuationDelimeters)
        {
            // Set these to small, fixed values for now.  We can be more
            // selective later, if we need more variation.
            punctuations = new string[] { "-", "'", "*" };
            punctuationChance = 10f;
            leadPunctuationChance = 35f;
        }

        dictionary = new Dictionary<string, string>();
        languageName = "";
        GenerateLanguageName();
    }

    /// <summary>
    /// Generates the language's actual name
    /// </summary>
    private void GenerateLanguageName()
    {
        // The "__" on the end will guarantee uniqueness, since all other
        // input sources are filtered for punctuation.
        languageName = TranslationForWordWithCase("LanguageName__");
    }

    /// <summary>
    /// Picks the number of syllables for a word (randomly)
    /// </summary>
    /// <returns></returns>
    private int SyllableCount()
    {
        if (percentSyllableCountChance.Length == 0) { return 2; }

        int chanceSum = 0;
        foreach (int chance in percentSyllableCountChance)
        {
            chanceSum += chance;
            Debug.Assert(chance > 0, "ASSERT: Zero or negative probability in percentSyllableCountChance.");
        }

        int num = UnityEngine.Random.Range(0, chanceSum);
        int outNum = 0;
        while (num > percentSyllableCountChance[outNum])
        {
            num -= percentSyllableCountChance[outNum];
            outNum++;
        }

        return outNum + 1;
    }

    /// <summary>
    /// Generates a word for the "baseLanguageWord" if it's not already present,
    /// then adds it to the dictionary and returns it.
    /// </summary>
    /// <param name="baseLanguageWord"></param>
    /// <returns></returns>
    public string GenerateWordFor(string baseLanguageWord)
    {
        if (!dictionary.ContainsKey(baseLanguageWord))
        {
            int numSyll = SyllableCount();
            string word = "";
            for (int i = 0; i < numSyll; i++)
            {
                word += syllables[UnityEngine.Random.Range(0, syllables.Count)];
                if (usesPunctuationDelimeters && i < (numSyll - 1))
                {
                    float percent = UnityEngine.Random.Range(0f, 100f);
                    if (percent < ((i == 0) ? leadPunctuationChance : punctuationChance))
                    {
                        int choice = UnityEngine.Random.Range(0, punctuations.Length);
                        word += punctuations[choice];
                    }
                }
            }

            dictionary[baseLanguageWord] = word;
        }

        return dictionary[baseLanguageWord];
    }

    public string TranslationForWordWithCase(string baseLanguageWord)
    {
        if (baseLanguageWord.Length == 0) { return baseLanguageWord; }

        string outWord = GenerateWordFor(baseLanguageWord.ToLower());

        // If the first character is uppercase, but the second one isn't, convert
        // the first character of the output word to upper case.
        if (char.IsUpper(baseLanguageWord, 0) &&
            !(baseLanguageWord.Length > 1 && char.IsUpper(baseLanguageWord, 1)))
        {
            outWord = char.ToUpper(outWord[0]) + outWord.Substring(1);
        }

        return outWord;
    }

    /// <summary>
    /// Parses the passed string into words (using spaces as delimeters),
    /// then calls GenerateWordFor on each one, concatenates the results,
    /// and returns the text.  
    /// </summary>
    /// <param name="baseLanguageText"></param>
    /// <returns></returns>
    public string GenerateTextFor(string baseLanguageText,
                  List<string> knownWords = null,
                  float percentGuess = 0f,
                  bool useBracketsForKnown = true)
    {
        string currentWord = "";
        string outString = "";

        foreach (char c in baseLanguageText)
        {
            if (char.IsLetter(c) || c == '\'')
            {
                currentWord += c;
            }
            else
            {
                if (currentWord.Length > 0)
                {
                    if (knownWords != null && knownWords.Contains(currentWord.ToLower()))
                    {
                        if (useBracketsForKnown)
                            outString += $"[{currentWord}]";
                        else
                            outString += currentWord;
                    }
                    else if (UnityEngine.Random.Range(0f, 100f) < percentGuess)
                    {
                        if (useBracketsForKnown)
                            outString += $"<{currentWord}>";
                        else
                            outString += currentWord;
                    }
                    else
                    {
                        outString += TranslationForWordWithCase(currentWord);
                    }
                    currentWord = "";
                }

                outString += c;
            }
        }

        // Repeat this check, just in case we didn't end with punctuation.
        if (currentWord.Length > 0)
        {
            outString += TranslationForWordWithCase(currentWord);
            currentWord = "";
        }

        return outString;
    }
}

```

