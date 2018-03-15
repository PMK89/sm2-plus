### sm2-plus extended interval
----

This is a JS implementation of a refined version of the SM2 space repetition learning algorithm invented by [**BlueRaja**][br] overcoming some of the inherent issues of the original version.
Details about what these issuses are and how [**BlueRaja**][br] solved them can be found in this [post][original].

The discribed algorithm has shown a maximal interval of 5 days even with a difficulty of 0 while other memory tools extend the interval to weeks or month. This can be prevented by adding to a dificulty weighted old interval instead of a constant.

#### Usage
```javascript
import { WORST, BEST, calculate, getPercentOverdue } from 'sm2-plus';

const DAY_IN_MINISECONDS = 24 * 60 * 60 * 1000;
const getDaysSinceEpoch = () => (
    Math.round(new Date().getTime() / DAY_IN_MINISECONDS)
);

const TODAY = getDaysSinceEpoch();

const testWord = {
  word: 'test',
  update: TODAY - 17,    
  difficulty: 0.2,
  interval: 100
};

console.info(calculate(testWord, BEST, TODAY));
```
The output should be:
``` javascript
{ 
  difficulty: 0.19,    
  interval: 1,
  dueDate: TODAY+1,
  update: TODAY,
  word: 'test' 
  }
```
Among them `dueDate` is the date the next time when the item should be reviewed.

#### Simulation
It is good to be able to make a evaluation about how many times does the user has to study to totally remember a word in the best case.

Thus a simulation method was export to do this job.

The first argument is the initial difficulty and the second argument is the threshold below which a word can be seen as remembered. 
```javascript
import { simulate } from 'sm2-plus'
simulate(0.3, 0.1);
```

The output would be like this, meaning that in the best condition, a user has to remember a word and select **BEST** 4 times to make sure that he has mastered the word whose initial difficulty is 0.3. 

| Day           | Index         | Difficulty   |
| ------------- | ------------- | -------------|
| 0             | 1             |  0.3                |
| 1             | 2             |  0.24117647058823527|
| 2             | 3             |  0.18235294117647055|
| 3             | 4             |  0.12352941176470585|
| 4             | 5             |  0.06470588235294114|

#### Algorithm Abstract

Each item should be stored as the following structure:
- difficulty:
   How difficult the item is, from [0.0, 1.0].  Defaults to 0.3 (if the software has no way of determining a better default for an item).
- interval: 
  How many days should occur between review attempts for this item.
- update:
  The last date this item was reviewed.

When reviewing item,  choose a performanceRating from [0.0, 1.0], with 1.0 being the best.  Set a cutoff point for the answer being “correct” (default is 0.6). Then the item data can be updated using the way described below:
- percentOverdue= (today - update) / interval) (2 <= percentOverdue)

- difficulty = difficulty + percentOverdue * (8 - 9 * performanceRating) / 17 (0 <= difficulty <= 1)

- difficultyWeight = 3 - 1.7 * difficulty

- interval =
  - Max(((1-difficulty)^3 * oldinterval), 1) + (difficultyWeight - 1) * percentOverdue (for correct answer)
  - 1 / difficultyWeight / difficultyWeight (for incorrect answer)


[original]:http://www.blueraja.com/blog/477/a-better-spaced-repetition-learning-algorithm-sm2
[br]:http://www.blueraja.com/blog/author/blueraja
