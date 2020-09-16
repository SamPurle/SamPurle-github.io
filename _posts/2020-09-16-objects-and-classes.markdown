---
layout: post
title:  "Objects and Classes"
date:   2020-09-16 08:45:55 +0100
categories: project upload
---

### Overview

This post will just be a collection of smaller scripts to showcase my learning process for picking up OOP.

### Guess The Number

This game could very easily be written as a simple function, but it's a good starting point to get used to implementing functionality within classes.

{% highlight python %}

import random

class Game():
    def __init__(self, n_guesses = 5):
        self.guesses_left = n_guesses
        self.number = random.randint(0, 50)
        print('Game time started. You have {} guesses remaining'.format(self.guesses_left))

    def guess(self):
        while self.guesses_left > 0:
            num = int(input('Guess your number: '))
            self.guesses_left -= 1

            if num == self.number:
                print('Congratulations! You won with {} guesses remaining!'.format(self.guesses_left))        
            elif num > self.number:
                print('Your guess was too high! You have {} guesses remaining'.format(self.guesses_left))
            else:
                print('Your guess was too low! You have {} guesses remaining'.format(self.guesses_left))      
        print('You lost lmao. The real number was {}'.format(self.number))     
        pass

my_game = Game()
my_game.guess()

{% endhighlight %}

### Noughts and Crosses

This is the first game that should probably be written as a class rather than a simple function. The UI isn't exactly pretty and the implementation of the win_check method is slightly clunky,
but both minima and maxima have to be checked taking [1] and [-1] as the two inputs.

{% highlight python %}

import numpy as np

class Grid:
    def __init__(self):
        self.grid = np.zeros_like(([0,0,0],[0,0,0],[0,0,0]))
        print(self.grid)
        self.i = -1
        self.won = 0
        while self.won == 0:
            row = int(input('Input your row: '))
            col = int(input('Input your col: '))
            self.move(row,col)    

    def move(self, r, c):
        self.i = -self.i
        self.grid[r-1][c-1] = self.i
        print(self.grid)
        self.win_check()

    def win_check(self):
        if (max(np.sum(a = self.grid, axis = 0)) == 3) or (min(np.sum(a = self.grid, axis = 0)) == -3):
            self.win()      
        if (max(np.sum(a = self.grid, axis = 1)) == 3) or (min(np.sum(a = self.grid, axis = 1)) == -3):
            self.win()
        if (np.trace(self.grid) == 3) or (np.trace(self.grid) == -3):
            self.win()
        flipgrid = np.fliplr(self.grid)
        if (np.trace(flipgrid) == 3) or (np.trace(flipgrid) == -3):
            self.win()

    def win(self):
        print('Player using {} wins!'.format(self.i))  
        self.won = 1

my_game = Grid()        

{% endhighlight %}

### Hangman

Hangman was one of the first games I built when learning python a few months ago, although at this point the game was played in the global scope through the use of a while loop. I'm very happy with how much cleaner this build is than the first time around, which shows how I'm approaching problems in a much more methodical way.

{% highlight python %}

import random

class Hangman:
    def __init__(self):
        words=open("dictionary.txt","r")
        wordslist = [line.split('\n') for line in words.readlines()]        
        self.real_word = random.choice(wordslist)[0].lower()
        self.length = len(self.real_word)
        self.guesses_remaining = 6
        self.blanks_list = ['_'] * self.length
        self.guessed_letters = []
        self.won = False

    def guess(self):
        while (self.guesses_remaining > 0) and (not self.won):
            print('You have {} guesses remaining!'.format(self.guesses_remaining))
            print(self.blanks_list)
            let = input('Please input your guess: ')
            if (let not in self.real_word) or (let in self.guessed_letters):
                    self.guesses_remaining -= 1
            for ind in range(len(self.real_word)):                
                if let == self.real_word[ind]:
                    self.blanks_list[ind] = let   
            self.guessed_letters.append(let)
            if '_' not in self.blanks_list:
                self.win()    
        if (not self.won):
            self.lose()

    def win(self):
        print('Congratulations! You correctly guessed the word "{}" with {} guesses remaining!'.format(self.real_word, self.guesses_remaining))
        self.won = True  

    def lose(self):
        print('You lost lol')


my_game = Hangman()
print(my_game.real_word)
my_game.guess()

{% endhighlight %}

### CSV Reader

This is just a simple class to show how they could be used for file input and output.

{% highlight python %}

import os

class CSVReader:
    def __init__(self, file_path):
        self.file_path = file_path
        self.columns = None
        self.data = None

    def read_csv(self):
        f = open(self.file_path, 'r')
        first_line = f.readline()        
        self.columns = []
        self.data = []
        for col in first_line.split(', '):
            self.columns.append(col.replace('\n', ''))
        for row in f.readlines():            
            mini_list = []
            for item in row.split(', '):
                mini_list.append(item.replace('\n', ''))
            self.data.append(mini_list)    
        f.close()

    def get(self, column_name, row_number):
        col_ind = self.columns.index(column_name)       
        return self.data[row_number][col_ind]


my_reader = CSVReader('{}/example_data.txt'.format(os.getcwd()))
my_reader.read_csv()
print(my_reader.get('occupation', 0))
print(my_reader.get('gender', 1))

{% endhighlight %}


### Self-playing Hangman

As a challenge to myself, I decided to see if I could build a version of Hangman that intelligently plays itself. I did this by creating a HangmanSolver class:

{% highlight python %}

class HangmanSolver:
    def __init__(self, blank_list):
        self.length = len(blank_list)
        self.unguessed = list(string.ascii_uppercase)
        print('Initiated solver')
        words = open("dictionary.txt","r")
        wordslist = [line.split('\n') for line in words.readlines()]   
        words.close()  
        self.posswords = []
        for item in wordslist:
            if len(item[0]) == self.length:
                self.posswords.append(item[0])  

{% endhighlight %}

This class is initialised with a blank_list from the Hangman class, and then determines the length of the word to be guessed. A list of all letters is created, which can be gradually removed from as guessing occurs and letters should not be re-guessed. A list of all words in the dictionary file that match the length of the target word is then created under self.posswords.

{% highlight python %}

def guess_select(self, blank_list):        
    notposs = set()           
    for word in self.posswords:            
        for index in range(self.length):                
            if (word[index] != blank_list[index]) and (blank_list[index] != '_'):                    
                notposs.add(word)                   
    for word in self.posswords:
        if word in notposs:
            self.posswords.remove(word)               
    count_dict = {}      
    for let in self.unguessed:
        let_count = 0            
        for word in self.posswords:
            for index in range(self.length):
                if (word[index] == let) and (blank_list[index] == '_'):
                    let_count += 1
        count_dict[let] = let_count          
    guess_let = max(count_dict, key = count_dict.get)
    self.unguessed.remove(guess_let)                  
    return guess_let

{% endhighlight %}

Instead of taking a user input method, I modified the Hangman class to call the guess_select() method from the HangmanSolver class, which takes in the list of blanks populated with correctly guessed letters from the Hangman class.

An empty set of dictionary words that do not match based on correctly guessed letters is then populated via a "for" loop. These are then programmatically removed from self.posswords. Note that I chose to add to a set and then subsequently remove from self.posswords, as I did not want to remove directly from self.posswords while iterating over it.  

Next, an empty dictionary is created ready to store the counts of each letter in potentially matching words is created. This is populated with letters that have not been guessed yet. For each letter, the number of times that letter occurs in an index corresponding to a blank space in the target word is counted. Note that this counts the number of **letter** occurrences, not the number of word occurences. This was deliberate, as more information is likely to be gained by correctly guessing a letter that appears many times in the word.

After all potential letters have been iterated over, the maximum count from the dictionary is found. The key corresponding to this maximum value is returned as the output argument, and passed to the Hangman.guess() method in place of a user input.

{:refdef: style="text-align: center;"}
<img src="{{site.url}}/{{site.baseurl}}/"assets\Classes\ezgif-6-816c4ae9b03c.gif"">
{: refdef}
