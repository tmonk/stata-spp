---
title: Advanced Programming 
author: T. Hall, T. Monk, J. Dittmar, LSE
weight: 600
---

{{% pageinfo color="info" %}}
This section provides a basic introduction to more advanced programming in Stata. We introduce macros, loops, and data frames which will allow you to use Stata more efficiently.
{{% /pageinfo %}}

## 6.1 Macros {#s61}

A macro is simply a name associated with some text. If you find yourself tying the same words over and over in Stata, a macro can probably help. Macros can be local or global. Local macros are ephemeral. When you work with do-file editor, local macros are valid only in a single execution of commands in do-files or ado-files. Global macros are persisting. Once defined whether in do-file editor or in interactive command window, they persist until you delete them or the session is ended.

A local macro is defined using the command `local name “text”` or `local name=expression`. You refer to a local macro using ``name’`.

A global macro is defined using `global name text` or `global name=expression`. You refer to a global macro using `$name`. To check the global macros in Stata’s memory use <code><u>ma</u>cro <u>di</u>rectory</code>.

**Practical Exercise: Creating a global macro for a set of regression control variables.**

To see how macros might be used in practice, suppose we need to run lots of regressions using the life expectancy data, all of which include a set of standard controls `safewater popgrowth c.safewater#c.safewater`. To avoid having to type these out each time, define the control variables in the global macro `$controls`.

```stata
. global controls "safewater popgrowth c.safewater#c.safewater"

. su $controls

    Variable |        Obs        Mean    Std. dev.       Min        Max
-------------+---------------------------------------------------------
   safewater |         40        76.1    17.89112         28        100
   popgrowth |         68    .9720588    .9311918        -.5          3
             |
 c.safewater#|
 c.safewater |         40      6103.3    2611.293        784      10000

. reg lexp gnppc $controls

```

Now as another example, to standardise the value of GNP per capita we subtract its mean and divide by the standard deviation. This can be easily achieved using local macros. Remember that local macros must be executed in a single go from a do file – they will not work if you run each command line separately.

**Practical Exercise: using local macros to standardise a variable.**

```stata
. sum gnppc

    Variable |        Obs        Mean    Std. dev.       Min        Max
-------------+---------------------------------------------------------
       gnppc |         63    8674.857    10634.68        370      39980

. local u = r(mean)

. local std = r(sd)

. g z_gnppc = (gnppc - `u')/`std'
(5 missing values generated)

```

## 6.2 Loops {#s62}

If you find yourself having to repeat the same task over and over, there is likely a clever way in Stata to automate what you are doing using loops. Think about the situation where you should replace some values of 1,000 variables the same way. You can do it manually, but it takes a lot of time and it is error-prone. Loops can be incredibly useful in these types of situations.

The command for looping over a sequence of numbers takes the form:

```
forvalues i = sequence {
… loop using `i’ …
}
```

Here `i` is the name of the local macro that will be set to each number in the sequence. Note that you don’t necessarily have to use `i` , you can define any letter/name you please. `sequence` is a range of numbers which can be:

- `min`/`max` to specify all the integers between min and max;
- `first(step)last` which yields a sequence from first to last in steps of size step. For example, 15(5)50 yields 15,20,25,30,35,40,45 and 50.

**Practical Exercise: Looping over a sequence of numbers.**

For example, to create dummy variables to represent 5-year age groups between 20 and 50:

```stata
. forvalues bot = 20(5)45 {
    local top = `bot' + 4
    gen age`bot'to`top' = age >= `bot' & age <= `top'
}
```

The second looping command is `foreach` and is used for looping over items in a list. The syntax for a foreach loop is:

```stata
. foreach i in list {
… loop using `i’ …
}
```

If you wanted to loop over an irregular sequence of numbers, you would need to use a foreach loop: 

**Practical Exercise: Looping over a list.**

For example, to create a new variable for GNP per capita in $00s or $000s:

```stata
. foreach i in 100 1000 {
     gen gnppc_by`i' = gnppc / `i'
  }
```
Foreach loops can be useful when used in conjunction with the `levelsof` command which identifies all the values in a variable and puts those values in a macro. The syntax for this command is: `levelsof variable, local (macroname)`. You can then refer to the list created by `levelsof` in a foreach loop, as in the example below where we summarise GNP per capita for each “level of” the region variable.

***Practical Exercise: Using foreach and levels of.***

Summarise GNP per capita for each region using a loop and levelsof.

```stata
. levelsof region, local(region_list)
1 2 3

. foreach i in `region_list' {    
  2.         su gnppc if region == `i'       
  3. }

    Variable |        Obs        Mean    Std. dev.       Min        Max
-------------+---------------------------------------------------------
       gnppc |         41    10738.05    11793.66        370      39980

    Variable |        Obs        Mean    Std. dev.       Min        Max
-------------+---------------------------------------------------------
       gnppc |         12    5817.167    8929.484        410      29240

    Variable |        Obs        Mean    Std. dev.       Min        Max
-------------+---------------------------------------------------------
       gnppc |         10        3645    2254.528       1010       8030

```

We can also use foreach loops to loop over words. 

```stata
. foreach word in I love Stata {
       display "`word'"
  }
I
love
Stata
```

## 6.3 Frames {#s63}
	
So far, we have been loading one dataset into Stata’s memory at a time. However, Stata can handle multiple datasets simultaneously using data frames, which work with the <code><u>frame</u>s</code> command. You can create frames, change between them, delete them and rename them with the following commands: `frame create framename`, `frame change framename`, `frame drop framename`,  `frame rename oldname newname`. 

**Practical Exercise: Using data frames in Stata.**

For a practical example, try creating a new frame with a subset of the life expectancy variables and switch between frames.

```stata
. sysuse lifeexp, clear
(Life expectancy, 1998)

. frames
  (current frame is default)

. frame rename default main

. frames put country gnppc lexp, into(subset)

. frames dir
  main    68 x 6; Life expectancy, 1998
* subset  68 x 3; Life expectancy, 1998

Note: Frames marked with * contain unsaved data.

. frame change subset

. g log_gnppc = log(gnppc)
(5 missing values generated)

. frame change main
```

Frames can be easily managed through the menus “Data -> Frames Manager”. There are many more applications of frames, type `help frames` to learn more.









