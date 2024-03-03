## AI Crossword Generator

This post is about a recent project of mine, an AI crossword generator. It started off as a course
project from a High Performance C Computing course, which involved a program that was able to fill
in an empty crossword grid with words from a provided wordlist. The initial implementation for this
part of the project was provided by [Professor Owens](https://www.ece.ucdavis.edu/~jowens/); for
the class portion of this project, I optimized the code which led to >100x speedup (varying based
the inputs). However, this implementation only had a very simple text interface and only output a
filled in board, but I wanted something a little more. In particular, my goal was to create a wrapper
around this code that would allow me to automatically generate a puzzle with clue's and output it
into a more presentable (non-ASCII) interface. I also wanted to be able to create themed crossword
puzzles, mainly for my sister who loves space and is an avid crossword puzzler(?). To do this, I
harnessed the power of ML and AI (call it whatever you want I guess).

### Using Embeddings to Create a Theme

To give my crosswords a theme, I enlisted word embeddings to help me preferrentially insert words into the puzzle. For reference, the backend generator uses words from a word bank; the words are chosen arbitrarily from the work bank to be placed into the puzzle (provided they fit with the other already place words). However, the modifications I made allowed words to be preferentially chosed based on their "word score", so we will attempt to place words with higher scores first.

For this project, I wanted words to have a higher score if they better fit into my "theme" for the puzzle; for instance, my theme could be "land" or "meat". Using word embeddings (every word is described as a vector of floats, derived from ML models), a word was given a higher word score if its embedding had a higher dot product similarity with the embedding of the theme word. Below, I show puzzle generated with themes of "meat" and "land". While we are able to see clear results on these sparser puzzles, the themes are less obvious on denser puzzles as the generator often chooses words with low scores just to attempt to fill the puzzle.

#### Meat

![Meat Crossword](/blog/images/meat_xword.png)


##### Land

![Land Crossword](/blog/images/land_xword.png)


### AI Clue Generation

With AI being all the rage, I wanted to see if I could integrate OpenAI's LLM 'GPT' into me project using their AI.  I tried a few test questions on ChatGPT and it was pretty good at making clues most of the time. However, there are lots of bloopers, especially since I'm not paying for access to GPT-4, but I like to think it gives it an adde challenge.

So with an API key and $5.00 of free credits in hand, I set up my program to interface with OpenAI() and ask it for crossword clues. The prompt I used is embeded in the code snippet below, I tried keeping the prompt short to save tokens. I might make a separate post on using the GPT API, but overall the Python interface is quite simple to use, though the documentation isn't as technical as one would hope.

```python
response = self.client.chat.completions.create(
    model="gpt-3.5-turbo-1106",
    messages=[
        {"role": "user",
         "content":
         f"write a {difficulty} cross word clue for "
         f"{solution_word} {f'{modifier} ' if modifier else ''}"
         "using less than 10 tokens"}
    ]
)
```

### Gallery

Here's a few example crosswords, feel free to try them out! Solutions are the bottom of the page.

(1)

![Rocket Crossword](/blog/images/rocket_board.png)
![Rocket Crossword (pt 2)](/blog/images/rocket_clues.png)

(2)

![Astronaut Crossword](/blog/images/astronaut_board.png)
![Astronaut Crossword (pt 2)](/blog/images/astronaut_clues.png)

### Solutions

(1)

![Rocket Crossword Solution](/blog/images/rocket_solution.png)

(2)

![Astronaut Crossword Solution](/blog/images/astronaut_solution.png)
