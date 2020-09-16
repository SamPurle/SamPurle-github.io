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
