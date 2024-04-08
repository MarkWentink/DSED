---
weight: 2
title: "A simple CPU: Advent of Code Throwback 2, Object-Oriented Programming"
date: 2023-11-24
draft: false
author: "M Wentink"
authorLink: "https://MarkWentink.github.io/DSED"
summary: "In this Advent of Code challenges, we take an object-orientated approach to programming a very simple CPU. "
images: []
resources:
- name: featured-image
  src: cpu.jpg
- name: featured-image-preview
  src: cpu.jpg
#categories: ["Advent of Code", "oop"]

lightgallery: true
---

# The most wonderful time of the year

It's almost December again, which means it's [Advent of Code](https://adventofcode.com/) time! AoC (not the US House Representative) is a series of daily coding challenges in December that can be solved in any language. I will code in Python throughout these posts.

In anticipation of all this excitement, I'm revisiting some of last year's problems, to give you a flavour of what the challenges involve, and hopefully to get you coding! 

Introducing: **AoC 2022 day 10**


# The challenge

As before, our first challenge is to extract the actual objective from the story, which fortunately for us, is a bit more to the point this time. 

[AoC 2022 Day 10](https://adventofcode.com/2022/day/10) asks us to program a very basic CPU: it has 
- a `clock` that counts up and keeps track of what cycle we're on, 
- a single `register` to store a number in, 
- and our CPU can execute a grand total of two operations:
  - `noop`, which does nothing for one cycle, and 
  - `addx` which takes two cycles to complete and adds a number to our register.

Our job is to run a series of instructions (lines of the puzzle input, either `noop` or `addx`) while periodically checking the value of the `register` and summing those values together into a `signal_strength`. 

(Btw, if you want to know more about how your computer's CPU works, [CrashCourse Computer Science](https://www.youtube.com/watch?v=FZGugFqdr60) has an excellent video on it!)





#### Rule summary

- Instructions are fed in the format `'noop'` or `'addx [nr]'`.
- The clock (starting at 1) counts cycles: +1 for a `noop` and +2 for a `addx`. 
- `noop` does nothing, i.e. the cpu just waits for one cycle. 
- `addx` changes the value of `register`, which starts at 1.
- On the 20th, 60th, 100th, 140th, 180th, and 220th cycle, we need to check the `register` value, multiply that to the `clock` value, and add the product to the running sum `signal_strength`.
- The answer is the total of `signal_strength`.


# Let's get coding

#### The plan

Because this challenge revolves around a central concept (the CPU), this is an ideal situation for some Object-Oriented Programming. 

We will create a `CPU` class, which has a clock and register attribute, and which can run operations as methods. 

Once we have the class, we will use our `load_data()` function from [before]({{< ref "string_checking" >}}) to read in the puzzle input, save that as a list of instructions, and work our way through the list. 

#### Keep it classy

As the centre piece for our program, we will create the `CPU` class. In object-oriented programming, classes are king. They get their own attributes (pieces of info about them), and methods (things they can do).

Classes are like blueprints, or templates. To use it, we create an 'instance' of the class, a copy, and that copy inherits all the properties of the template. 

Let's create our class, and then zoom in on some specific pieces.

```python
class CPU():
    
    # Every class must have an __init__ method. 
    # This defines what happens when the variable is first created. 
    # In our case, the cpu gets three 'attributes', with pre-set values. 
    def __init__(self):
        self.register = 1
        self.clock = 1
        self.signal_strength = 0
        return
    
    
    # triggering a noop instruction should increment the internal clock of the cpu
    # It should then test whether a cycle 20, 60, 100, ... is reached.
    def noop(self):
        self.clock += 1
        self.test()
        return
    
    
    # Triggering an add instruction should update the clock as well as the register.
    # We also need to test whether a signal cycle is reached after each increment
    def addx(self, increment):
        self.clock += 1
        self.test()
        
        # Note the register is only updated after the second cycle increment.
        self.clock += 1
        self.register += increment
        self.test()
        return
    
    
    # Our test method checks whether the clock has reached a signal cycle (20, 60, 100, ...)
    # If so, it calculates the signal strength and adds that to the running total. 
    def test(self):
        if self.clock % 40 == 20:
            self.signal_strength += self.clock * self.register
        return 
```

Woosh, that's quite the chunk! 

Let's break that up a bit:

#### The `__init__` method

```python
    def __init__(self):
        self.register = 1
        self.clock = 1
        self.signal_strength = 0
        return
```

A class definition always needs an `__init__()` method. This specifies the pieces of info that need to be assigned when a copy of the template is requested ('instantiated'). 

We can make the copy by just treating the class as a function and assigned the output to a variable: `my_cpu = CPU()`

We would then have three variables, `my_cpu.clock`, `my_cpu.register`, and `my_cpu.signal_strength`, all with pre-set values.

Don't worry too much about the whole `self.something` syntax in the definition. This is a way for our class to refer to information about itself. 

Ok, at this point, we have a cpu with some pieces of info, but it doesn't do anything yet. Our CPU needs to be able to do three things: deal with `noop` instructions, deal with `addx` instructions, and check whether the clock is on a signal cycle (20, 60, 100, 140, 180, 220).

Let's make that happen:

#### Noop()

```python
    def noop(self):
        self.clock += 1
        self.test()
        return
```

The only thing a `noop()` instruction does is let the clock tick. This essentially corresponds to a 'wait' or 'hold' command to the cpu. 

At the end of every instruction, we want to check whether we're on a signal cycle. We'll come to `test()` in a moment. 

(Again, class methods always take `self` as an input, and all the attributes are referred to as `self.something`. Without the self, you get class attributes, which are a story for another time.)

#### addx()

```python
def addx(self, increment):
        self.clock += 1
        self.test()
        
        self.clock += 1
        self.register += increment
        self.test()
        return
```
`addx` takes two cycles to complete, so the clock gets incremented twice. 

After the first increment, nothing has changed yet, but we still need to check whether we landed on a signal cycle. 

After the second increment, the register is updated by adding the value that's part of the instruction. We then check again for a signal cycle. 

> One beautiful thing about classes is that although all these methods might seem like indepedent functions, they all share an awareness of the self. attributes. This means that we often don't need to return anything at the end of a method. The attributes are automatically updated. 

#### Lastly, our test()

```python
def test(self):
    if self.clock % 40 == 20:
        self.signal_strength += self.clock * self.register
    return 
```

`test()` should only trigger on cycles 20, 60, 100, 140, 180, and 220. A nice shorthand for this list uses the modulus function, and the fact that these points are 40 cycles apart. If the remainder of clock divided by 40 is 20, then we're on a signal cycle. 

In that case, we need to multiply the clock value with the register value, and add the result to a running total, `signal_strength`.

#### Putting it all together

Now that we have our `CPU()` class, all we need is a `load_data()` function and a tiny bit of code.

I'm reusing the same `load_data()` function as for the previous challenge, with the addition of a `.split()` to separate the `addx` command from the integer value. 

```python
def load_data(filepath):
    data = []
    with open(filepath) as f:
        for line in f.readlines():
            data.append(line.strip().split())
    return data
```
This returns the puzzle input as a list of instructions. Here are the first five:

`[['addx', '1'], ['noop'], ['addx', '4'], ['noop'], ['noop']]`

Note `noop` instructions come by themselves, while `addx` instructions come with an integer value.


Our actual program then looks like:

```python
# Pull in the data
data = load_data(filepath)
# Ask for a copy of the CPU template
my_cpu = CPU()


# Loop through the instructions
for instruction in data:
  
  # First check that we haven't reached cycle 220 yet.
  # If we have, interrupt and stop the loop. 
  if cpu.clock >= 221:
        break


  # Depending on the command, route to the noop() method or the addx() method
  if instruction[0] == 'noop':
    my_cpu.noop()

  # we need to convert the input from string to integer before passing it to the method.        
  elif instruction[0] == 'addx':
    my_cpu.addx(int(instruction[1]))

  # as a safeguard, we'll build in an error message for any odd instructions.        
  else:
    print("Error: instruction not recognised")


# At the end of the loop, we simply print out our total signal strength as answer
print(my_cpu.signal_strength)
```

After all that comes rolling out the answer: `11820` in my case. This will be different for everyone though, as AoC generates unique puzzle input for every participant. Again, we are rewarded with a bright gold star, and access to the second half of the problem...

#### Wrap up

If Object Oriented Programming is new to you, don't let that put you off! Of course, you could solve this challenge just fine with normal functions, but often these challenges are good opportunities to try something new. You could even challenge yourself to tackle a whole year in a language that's new to you!

If you want to explore OOP more, I recommend the [W3School page](https://www.w3schools.com/python/python_classes.asp) on the topic, which does an excellent job of explaining the topic from scratch. 

See you next time!

-------

Got questions? Spotted any issues in the code? Or do you want to share your own examples of implementations? Drop me a message on [LinkedIn](https://www.linkedin.com/in/mark-wentink-793217116/).

-------

Photo Credit: Harrison Broadbent, Unsplash