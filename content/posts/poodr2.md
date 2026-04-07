---
title: 'Chapter 2 - Designing Classes with a Single Responsibility'
date: '2026-04-07T09:37:26+05:30'
draft: false
tags: ["poodr", "oop", "book"]
showToc: true
weight: 1
---

## Grouping Methods into Classes

The classes you create will affect how you think about your application forever. They define a virtual world, one that constrains the imagination of everyone downstream.

Despite the importance of correctly grouping methods into classes, at this early stage of your project you cannot possibly get it right.

>Design is more the art of preserving changeability than it is the act of achieving perfection.

## Organizing Code to Allow for Easy Changes

We define *easy to change* as:
- changes have no unexpected side effects, (**transparent**)
- small changes in requirements require correspondingly small changes in code (**reasonable**)
- existing code is easy to reuse, and (**usable**)
- the easiest way to make a change is to add code that in itself is easy to change (**exemplary**)

## An Example Application: Bicycles and Gears

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @rim = rim
    @tire = tire
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    # tire goes around rim twice for diameter
    ratio * (rim + (tire * 2))
  end
end

puts Gear.new(52, 11, 26, 1.5).gear_inches
# => 137.0909090

puts Gear.new(52, 11, 24, 1.25).gear_inches
# => 125.272727
```

The `gear_inches` assumes that rim and tire sizes are given in inches.

## Why Single Responsibility Matters

Applications that are easy to change consists of classes that are easy to reuse.

A class that has more than one responsibility is difficult to reuse. The various responsibilities are likely thoroughly entangled *within* the class. If you want to reuse some (but not all) of its behavior, it is impossible to get only the parts you need. You are faced with two options and neither is particularly appealing.

More responsibility means many reasons to change. Each time it changes, there's a possibility of breaking every class that depends on it.

## Determining If a Class Has a Single Responsibility

Re-phrasng methods of a class ought to make sense.
Ex: 
1. "Mr.Gear what is your ratio" (reasonable)
2. "Mr.Gear what are your gear inches" (shaky ground)
3. "Mr.Gear what is your tire (size)?" (ridiculous)

Attempt to describe what a class is doing in one sentence. If it contains words like "and", "or" then it might have more than one responsibility.

OO designers use the word *cohesion* to describe this concept. When everything a class does is highly related to its purpose, the class is said to be cohesive.

Q: How would we describe the responsibility of the **Gear** class?
A: "Calculate the effect that a gear has on a bicycle" (gear_inches is now on solid ground but tire size is still shaky)

The class doesnt' feel right. Gear has more than one responsibility, but it's not obvious what should be done.

## Determining When to Make Design Decisions

Do not feel compelled to make design decisions prematurely. Resist, even if you fear your code would dismay the design gurus.

Always ask yourself, "*What is the future cost of doing nothing today?*". If the answer is no, then don't make a design decision.

Conveniently, the new requirements will supply the exact information you need to make good design decisions.

The structure of every class is a message to future maintainers of the application. It reveals your design intentions. For better or worse, the patterns you establish today will be replicated forever.

**Gear** *lies* about your intentions. It is neither *usable* nor *exemplary*. It has multiple responsibilities and so should not be reused. It is not a pattern that should be replicated.

>"Improve it now" vs "improve it later" tension always exists. Applications are never perfectly designed. Every choice has a price.


## Writing Code That Embraces Change

### Depend on Behavior, Not Data

**Hide Instance Variables** : Always wrap instance variables in accessor methods instead of directly referring to variables. You should hide data from yourself. Doing so protects the code from being affected by unexpected changes. 

Data very often has behavior that you don't yet know about. Send messages to access variables even if you think of them as data.

**Hide Data Structures** : If being attached to an instance variable is bad, depending on a complicated data structure is worse.
```ruby
class ObscuringReferences
  attr_reader :data
  def initialize(data)
    @data = data
  end
  
  def diameters
    #0 is rim, 1 is tire
    data.collect { |cell| cell[0] + (cell[1] * 2)}
  end
  # ... many other methods that index into the array
end
```
This class expect to be initialized with a 2D array of rims and tires:
```ruby
# rim and tire sizes (in mm) in a 2d array
@data = [[622, 20], [622, 23], [559, 30], [559, 40]]
```

The `data` method merely returns the array. To do anything useful, each sender of `data` must have complete knowledge of what piece of data is at which index in the array.

Direct references into complicated data structures are confusing, because they obscure what the data really is, and they are a maintenance nightmare, because every reference will need to be changed when the structure of the array changes.

To separate structure from meaning in Ruby, use `Struct` class to wrap a structure.

```ruby
class RevealingReferences
  attr_reader :wheels
  def initialize(data)
    @wheels = wheelify(data)
  end
  
  def diameters
    wheels.collect { |wheel| wheel.rim + (wheel.tire * 2) }
  end
  
  # now everyone can send rim/tire to wheel
  Wheel = Struct.new(:rim, :tire)
  def wheelify(data)
    data.collect { |cell| Wheel.new(cell[0], cell[1]) }
  end
end
```

The `wheelfiy` method above isolates the messy structural information and **DRYs** out the code. It makes this class far more tolerant of change.

Although it might be easier to just have an array of Wheels to begin with, it is not always possible. If you can control the input, pass in a useful object, but if you are compelled to take a messy structure, hide the mess even from yourself.

### Enforce Single Responsibility Everywhere

1. **Extract Extra Responsibilities from Methods**
Just like classes methods should also follow single responsibility principle.
```ruby
def gear_inches
  ratio * diameter
end

def diameter
  rim + (tire * 2)
end
```

Gear's diameter method depends solely on things in Wheel. This suggest that method ought to be in Wheel. Gear is definitely responsible for calculating `gear_inches`, but Gear should not be calculating wheel diameter.

2. **Isolating Extra Responsibilities in Classes**

Should we isolate Wheel like behavior from Gear into its own class?

You will be best served by postponing decisions until you are absolutely forced to make them. Any decision you make in advance of an explicit requirement is just a guess. Don't decide preserve your ability to make a decision *later*.
```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @wheel = Wheel.new(rim, tire)
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end

  Wheel = Struct.new(:rim, :tire) do
    def diameter
      rim + (tire * 2)
    end
  end
end
```

Now you have a `Wheel` that can calculate its own diameter. Embedding this Wheel in Gear is obviously not the long-term design goal; it's more an experiment in code organization. It cleans up `Gear` but defers the decision about `Wheel`.

##  Finally, the Real Wheel

Embedding Wheel inside of Gear suggests that you expect that a Wheel only exist inside the context of Gear, common sense suggests otherwise. In this case, enough information exists right now to support the creation of an independent Wheel class. However, every domain isn't this clear cut.

Then the new requirement for calculating "bicycle wheel circumference" came. The real change now, is that your application has an explicit need for a `Wheel` that it can use independently of `Gear`. Its time to set `Wheel` free to be a separate class of its own.

Because you have already carefully isolated the `Wheel` behavior inside of `Gear` class, this change is painless.

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(chainring, cog, wheel=nil)
    @chainring = chainring
    @cog = cog
    @wheel = wheel
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end
end

class Wheel
  attr_reader :rim, :tire

  def initialize(rim, tire)
    @rim = rim
    @tire = tire
  end

  def diameter
    rim + (tire * 2)
  end

  def circumference
    diameter * Math::PI
  end
end

wheel = Wheel.new(26, 1.5)
puts wheel.circumference
# => 91.06

puts Gear.new(52, 11, wheel).gear_inches
# => 137.090

puts Gear.new(52, 11).ratio
```

---

## Summary

Classes that do one thing *isolate* that thing from the rest of your application. This isolation allows change without consequence and reuse without duplication.