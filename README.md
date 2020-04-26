# Keyboard layout evaluation

An evaluation of existing keyboard layouts over multiple languages, focused on ergonomic keyboards.

Many keyboard layouts exist (designed by hand or generated by algorithm) to improve on the ergonomics of Qwerty. However, they are typically assessed for typing in a single language, and on a standard keyboard. This analysis evaluates those layouts over several languages, and for an ergonomic keyboard.

The method uses statistics of bigram use (sets of 2 letters) for each language, and grades them according to subjective "weights" (depending on the keys positions), to calculate a comparative difficulty between layouts.

The results show that any alternative layout gives a significant ergonomic advantage over Qwerty. The best option is [MTGAP 2.0](http://mtgap.bilfo.com/completed_keyboard.html), but the [Colemak DHm](https://colemakmods.github.io/mod-dh/keyboards.html) performs very well and brings more familiarity, better shortcuts, and larger positive user feedback.

![results](images/results.png "Grades per layout")

# Table of contents

- [Keyboard layout evaluation](#keyboard-layout-evaluation)
- [Table of contents](#table-of-contents)
- [Character statistics](#character-statistics)
  - [count.py](#countpy)
  - [Spreadsheet analysis](#spreadsheet-analysis)
    - [Character counts](#character-counts)
    - [Bigram counts](#bigram-counts)
  - [Punctuation](#punctuation)
  - [Takeaway](#takeaway)
- [Layout evaluation](#layout-evaluation)
  - [Focus definition](#focus-definition)
  - [Evaluation principle](#evaluation-principle)
    - [Key base weights](#key-base-weights)
    - [Penalties](#penalties)
  - [Limits](#limits)
    - [Bigram frequencies variations](#bigram-frequencies-variations)
    - [Accented characters](#accented-characters)
  - [script.py](#scriptpy)
  - [Results](#results)
- [Conclusion](#conclusion)

# Character statistics

Contained in folder `character_stats`.

The [layout evaluation](#layout-evaluation) needs bigram frequencies (sets of 2 letters) for each language.

The frequencies are sourced from [Practical Cryptography](http://practicalcryptography.com/cryptanalysis/letter-frequencies-various-languages/) for English, French, Spanish, and German; and from [Norvig](http://norvig.com/mayzner.html) for English (to compare).

For comparison my own corpus is also analyzed (for English and French); made of my emails, some texts from free books, and some internet articles.

## count.py

Requirements: Python 3, Pandas.

The script `count.py` takes the text files in the `data` folder and outputs the character counts in `chars.csv`, and the bigram counts in `bigrams.csv`. Upper case are converted to lower case.

The list of characters to take into account is configurable in the code, in the list `chars`. Currently it takes the basic alphabet, plus `éèêàçâîô.,-'/`. 

The provided `chars.csv` and `bigrams.csv` files were generated with a personal corpus of emails (`mails_en` and `mails_fr`, 300\~400kB of raw text each) and various free books and articles (`vrac_en` and `vrac_fr`, 200\~400kB each).

## Spreadsheet analysis

This analysis is done in the Libreoffice spreadsheet `stats.ods`.

### Character counts

The characters frequencies for both English and French are quite consistent between the sources and my own corpus.

![chars_en](images/chars_en.png "Character occurences in corpus, English")

[Here](images/chars_fr.png) is the same chart for French.

### Bigram counts

The bigram counts show more discrepancies however. The charts below show the top 80 bigrams (sorted by average of both sources for English).

![bigram_en](images/bigram_en.png "Bigram occurences in corpus, English")

[Here](images/bigram_fr.png) is the same chart for French.

## Punctuation

The "theory" numbers from Practical Cryptography and Norvig do not contain data on punctuation characters such as `.,-'/`. However while the use of `.` and `,` depend a lot on each author's style, taking into account a non-zero frequency is essential due to their frequency.

The relevant bigram frequencies are not present in any existing statistics, but for French [the Bépo layout project](https://bepo.fr/wiki/Fr%C3%A9quence_des_caract%C3%A8res) gives character frequencies from a Wikipedia 2008 dump.

For English, [Vivian Cook's analysis](http://www.viviancook.uk/Punctuation/PunctFigs.htm) gives frequencies by word. [This University of Maryland paper](http://www.cs.umd.edu/hcil/trs/2013-17/2013-17.pdf) (Table 1) gives a frequency for the period of 1.151% (from Google Ngram data).

The frequencies from the personal corpus contain those characters so it can be used as a control. However it is useful to have more realistic "theory" results. The approach is to "fix" the missing frequencies by copying the relevant ones from the personal corpus, and normalizing the column.

To check that those inputs make sense, they can be compared to the few data available.

For English, we can estimate by dividing the frequencies from Vivian Cook by the average word length of 4.79 letters (from Norvig).

| For English | Personal corpus | Frequency / Word length | Google Ngram |
| :---------- | --------------: | ----------------------: | -----------: |
| Period .    |           1.6 % |                  1.36 % |      1.151 % |
| Comma ,     |           1.2 % |                  1.29 % |              |

| For French | Personal corpus | Wikipedia dump |
| :--------- | --------------: | -------------: |
| Period .   |           1.2 % |         0.83 % |
| Comma ,    |           1.5 % |         1.02 % |

Obviously some variation can be observed, but the numbers from the personal corpus pass that sanity check. It seems a bit more punctuation is used compared to the average literature, which isn't bad to consider.

## Takeaway

For the evaluation, the "theory" numbers will be the average from both sources for English, and the only source I have for other languages. But the differences with my own corpus show the sensitivity of those inputs, therefore the results should be taken with some tolerance.

As the "theory" numbers do not contain characters such as `.,-'/`, those will be copied from the personal corpus data then normalized.

# Layout evaluation

Contained in folder `layout_evaluation`.

## Focus definition

The influence of the physical keyboard is on the weights and penalties. The algorithm is more or less the same otherwise.

The chosen weights are for an ergonomic keyboard, which is why they are symmetrical. It would be similar for any non-staggered keyboard (meaning no horizontal shift between rows).

Only the keys on the main "matrix" are taken into account (3×12 matrix), which are reachable by a finger easily. The "numbers" row is ignored as we focus on the alphabetical layout.

Thumb keys available on ergonomic keyboards are ignored as I arbitrarily prefer not to place any alphanumeric character on them.

![layout_matrix](images/layout_matrix.svg "Keys names on matrix layout")

The keys are designated by a code (hand, row, column). The numbering includes space for some keys unused in the current script, in case of evolution (for instance for [keys on an Ergodox-like keyboard](images/layout_ergodox.svg)).

## Evaluation principle

For each language, the bigram frequencies are imported from the characters statistics, as percentage of use.

The principle is to assign a difficulty (weight) to a bigram (two keys typed consecutively). This weight can be multiplied by the bigram frequency, and all the bigram results summed up to get a general difficulty value of the layout.

Weight<sub>layout</sub> = sum( Weight<sub>bigram</sub> × Probability<sub>bigram</sub> )

The bigram weight is composed of:
- The weights assigned to the two key, representing the relative difficulty to push them
- A penalty, representing the added difficulty of pushing those 2 keys one after the other.

Weight<sub>bigram</sub> = Weight<sub>key1</sub> + Weight<sub>key2</sub> + Penalty<sub>key1 & key2</sub>

The results for all layouts and languages are finally normalized compared to Qwerty in English (at 100%).

### Key base weights

The base weights are shown below. The home row is identified in red border. 

They represent the relative difficulty to hit a single key. The proposed values are for an ergonomic keyboard, with vertical columns and a comfortable home row position.

![weights](images/weights.png "Keys weights")

### Penalties

The penalties represent the additional difficulty of to hitting 2 keys consecutively. They come on top of the base weight. They are only taken into account if the 2 keys are hit by the same hand (and are not the same key, like "aa").

Generally the penalties are higher if the same finger is used, and also the more rows separate the 2 keys. They can be negative if the relative position makes the motion easy, such as a close "inward roll" (like "sd" on Qwerty).

| First finger | Second finger | Same row | 1 row jump | 2 rows jump | Comment     |
| :----------- | :------------ | -------: | ---------: | ----------: | :---------- |
| Index        | Index         |      2.0 |        3.0 |         4.0 | Same finger |
| Index        | Middle        |      0.0 |        0.5 |         1.5 |
| Index        | Ring          |      0.0 |        0.3 |         1.0 |
| Index        | Pinky         |      0.0 |        0.3 |         0.6 |
| Middle       | Index         |     -2.0 |       -0.8 |         1.0 | Inward roll |
| Middle       | Middle        |      N/A |        3.0 |         4.0 | Same finger |
| Middle       | Ring          |      0.0 |        0.5 |         1.5 |
| Middle       | Pinky         |      0.0 |        0.3 |         1.0 |
| Ring         | Index         |     -2.0 |       -0.8 |         0.8 | Inward roll |
| Ring         | Middle        |     -2.5 |       -1.0 |         0.8 | Inward roll |
| Ring         | Ring          |      N/A |        3.0 |         4.0 | Same finger |
| Ring         | Pinky         |      0.5 |        1.0 |         2.0 |
| Pinky        | Index         |     -2.0 |       -0.8 |         0.2 | Inward roll |
| Pinky        | Middle        |     -2.0 |       -0.8 |         1.0 | Inward roll |
| Pinky        | Ring          |     -2.0 |       -0.8 |         2.0 | Inward roll |
| Pinky        | Pinky         |      2.5 |        3.5 |         5.0 | Same finger |

## Limits

### Bigram frequencies variations

As stated in the character statistics, the results are approximate as the bigram frequencies aren't a precise and objective number for everyone. 

However the results for [English](images/results_en.png) and [French](images/results_fr.png) show very little difference between the "theory" values and my personal corpus. Therefore it seems the variation in bigram use doesn't affect the final grade very significantly.

### Accented characters

The results for languages outside English are slightly off because most accented characters are not taken into account. 

Currently the ignored characters are `êàçâîôñäöüß/`, mainly because those characters are absent from most considered layouts. The characters `é` and `è` were added manually to the layouts (on unused keys, on vowel side if there's one) because I particularly care about French, and due to their high frequency (2.85%).

The characters `'` and `-` were also added when missing, on unused keys.

The issue mainly affects German (`äöüß`, 1.56% of the characters), but also French (`êàçâîô`, 0.75%) and Spanish (`ñ`, 0.22%).

To mitigate this effect the bigram frequencies are normalized after removing the ignored characters, so the summed grade is still calculated over 100%.

## script.py

Requirements: Python 3, Pandas, Matplotlib.

The script `script.py` uses the bigram statistics from `stats.csv`, and the configuration (key weights, penalties, and layouts definitions) from `config.txt` to generate the results (table and plot).

To customize the script, edit `config.txt` and have a look at the `main()` function.

The code isn't very efficient as it iterates through dataframes to generate the results. It executes in \~10s so in practice it doesn't really matter.

## Results

Here are the full results, for all 4 languages and including my personal corpus for English and French.

![results](images/results_full.png "Grades per layout")

In addition, here are the results for [only English](images/results_en.png) and [only French](images/results_fr.png). For comparison this includes the results before adding the punctuation as described [earlier](#punctuation). The necessity of taking into account the punctuation is clear. Both the "theory" and personal corpus give very similar grades, so results are quite consistent.

Here is the complete results list. The layouts can be seen in `config.txt`.

| Layout                                                                            | English | English perso | French | French perso | Spanish | German |
| :-------------------------------------------------------------------------------- | ------: | ------------: | -----: | -----------: | ------: | -----: |
| [MTGAP 2.0](http://mtgap.bilfo.com/completed_keyboard.html)                       |   61.42 |         60.80 |  64.05 |        62.50 |   60.10 |  61.58 |
| MTGAP 2.0 mod                                                                     |   61.45 |         60.83 |  62.52 |        61.32 |   60.10 |  61.58 |
| [MTGAP "ergonomic"](http://mtgap.bilfo.com/official_keyboard.html)                |   62.26 |         62.26 |  66.81 |        66.27 |   61.42 |  63.31 |
| Colemak DHm mod                                                                   |   62.50 |         62.21 |  64.88 |        64.02 |   61.12 |  63.33 |
| [MTGAP](https://mathematicalmulticore.wordpress.com/the-keyboard-layout-project/) |   62.53 |         61.38 |  67.66 |        66.97 |   63.56 |  64.29 |
| [Colemak DHm](https://colemakmods.github.io/mod-dh/keyboards.html)                |   62.62 |         62.34 |  65.30 |        64.35 |   61.12 |  63.33 |
| [MTGAP "shortcuts"](http://mtgap.bilfo.com/official_keyboard.html) (ROTS)         |   63.65 |         63.28 |  66.58 |        65.75 |   60.54 |  62.41 |
| [Workman](https://workmanlayout.org/)                                             |   63.66 |         63.36 |  70.41 |        69.35 |   65.32 |  65.03 |
| [MTGAP "standard"](http://mtgap.bilfo.com/official_keyboard.html)                 |   63.85 |         63.25 |  66.35 |        65.25 |   61.53 |  64.27 |
| [MTGAP "Easy"](http://mtgap.bilfo.com/official_keyboard.html)                     |   64.05 |         63.69 |  66.54 |        65.12 |   61.55 |  62.12 |
| [Colemak DH](https://colemakmods.github.io/mod-dh/)                               |   64.07 |         63.69 |  67.30 |        66.35 |   63.17 |  64.35 |
| [Kaehi](https://geekhack.org/index.php?topic=98275.0)                             |   64.20 |         62.94 |  68.37 |        67.25 |   64.43 |  66.20 |
| [Colemak](https://colemak.com/)                                                   |   64.53 |         64.41 |  65.65 |        64.14 |   61.94 |  66.03 |
| [Oneproduct](https://geekhack.org/index.php?topic=67604.0)                        |   65.29 |         65.03 |  73.11 |        72.12 |   67.43 |  67.46 |
| [Norman](https://normanlayout.info/)                                              |   65.98 |         65.46 |  72.30 |        70.74 |   68.93 |  67.84 |
| [ASSET](http://millikeys.sourceforge.net/asset/)                                  |   66.73 |         66.21 |  67.38 |        65.55 |   64.01 |  68.88 |
| [BEAKL](https://deskthority.net/wiki/BEAKL)                                       |   66.84 |         65.94 |  71.34 |        70.39 |   67.68 |  71.82 |
| [qgmlwyfub](http://mkweb.bcgsc.ca/carpalx/?full_optimization)                     |   69.13 |         68.66 |  74.43 |        74.89 |   69.17 |  70.49 |
| [Qwpr](https://sourceforge.net/projects/qwpr/)                                    |   69.15 |         68.83 |  70.42 |        69.90 |   65.88 |  69.67 |
| [Carpalx](http://mkweb.bcgsc.ca/carpalx/?full_optimization)                       |   69.44 |         69.22 |  74.89 |        75.38 |   69.79 |  72.91 |
| [Minimak-8key](http://www.minimak.org/)                                           |   70.69 |         70.15 |  72.57 |        71.01 |   69.10 |  72.77 |
| [Bépo 40%](http://bepo.fr/)                                                       |   70.99 |         71.30 |  66.45 |        64.97 |   66.83 |  70.41 |
| [Bépo keyberon](https://github.com/TeXitoi/keyberon#whats-the-layout)             |   71.25 |         71.28 |  66.67 |        65.18 |   66.84 |  70.45 |
| [Coeur](https://bepo.fr/wiki/Utilisateur:Bibidibop)                               |   72.72 |         73.12 |  66.64 |        64.97 |   66.44 |  70.80 |
| [Dvorak](https://en.wikipedia.org/wiki/Dvorak_keyboard_layout)                    |   73.54 |         72.82 |  77.90 |        75.72 |   75.04 |  74.05 |
| [Neo](https://neo-layout.org/)                                                    |   74.48 |         73.68 |  74.85 |        73.77 |   72.63 |  70.24 |
| Qwertz                                                                            |   99.02 |         98.07 |  98.26 |        97.60 |   92.73 |  98.26 |
| Qwerty                                                                            |  100.00 |         99.14 |  98.97 |        98.64 |   91.16 |  99.45 |
| Azerty                                                                            |  107.19 |        106.36 | 105.02 |       104.68 |  102.95 | 103.40 |

As the "MTGAP 2.0" and "Colemak DHm" layouts gave the most interesting results, a modified version of each was added to replace some characters like `;` (that can be on a layer like Shift + `,`) by the French `é`.

# Conclusion

All the alternative layouts perform very significantly better than the traditional ones (Qwerty and similar). The biggest interest for ergonomics is using an alternative layout at all, the selected choice matters a lot less.

![results](images/results.png "Grades per layout")

Among all options, the ones generated by [Mathematical multicore](https://mathematicalmulticore.wordpress.com/) (MTGAP) perform the best. In particular the MPGAP 2.0 (below) gives excellent results across all 4 languages. This simple analysis really shows the quality of the MTGAP approach.

![MTGAP 2.0](images/MTGAP20.png "MTGAP 2.0 layout")

In parallel, the Colemak variants give very decent results without compromising any of the 4 languages. The Colemak DHm (below) is found to be particularly adapted, which makes sense as it stems from user feedback on ergonomic keyboards.

![Colemak DHm](images/ColemakDHm.png "Colemak DHm layout")

In conclusion, the most optimized layout is found to be MTGAP 2.0, but the Colemak DHm is particularly recommended due to better keyboard shortcuts access, familiarity with Qwerty, and larger positive user feedback. 
