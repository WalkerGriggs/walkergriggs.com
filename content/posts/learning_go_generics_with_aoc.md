+++
title = "Learning Go Generics with Advent of Code"
author = ["Walker Griggs"]
date = 2021-12-15
categories = ["devlogs"]
draft = false
creator = "Emacs 27.1 (Org mode 9.6 + ox-hugo)"
weight = 2006
+++

_This post is a living draft and may be revised. If you have any comments, questions, or concerns, please reach out._

Yesterday, the Go core team released [go1.18beta1](https://go.dev/blog/go1.18beta1) which formally introduces generics. There isn't a whole lot of info circulating yet aside from git history and [go-nuts](https://groups.google.com/g/golang-nuts) experiments, but the overall reception feels very positive.

Personally, I've been hands on with generics for the better part of a week all thanks to the [Advent of Code](https://adventofcode.com), which has been the perfect venue to take generics for a spin. If you're not familiar with AOC...

> Advent of Code is an advent calendar of small programming puzzles for a variety of skill sets and skill levels that can be solved in any programming language you like. People use them as a speed contest, interview prep, company training, university coursework, practice problems, or to challenge each other.
>
> You don't need a computer science background to participate - just a little programming knowledge and some problem solving skills will get you pretty far. Nor do you need a fancy computer; every problem has a solution that completes in at most 15 seconds on ten-year-old hardware. -- [Eric Wastl](http://was.tl/)

This article will cover the basics of generics (or enough to get you started) and uses my AOC experiments a case study.


## Generics, generally {#generics-generally}

Go feels immediately more flexible with generics. The language is less prescriptive but still opinionated, and the implementation feels wonderfully idiomatic. But what do I mean by that?

For starters, generics feel very low-touch from a developers point of view. They've only added three new features:

-   Type parameters for functions and types
-   Type sets defined by interfaces
-   Type inference


### Type parameters {#type-parameters}

Type parameters are one or more name-type parings that look visually similar to our standard parameters; the only difference being type params are surrounded by square brackets, not parentheses. The square brackets, thankfully, are a consistent syntax you'll see used in struct declarations and variable initialization.

```go
[a, b constraint1, c constraint2]
```

Consider the `Max` function you've written dozens of times. We can now replace our strongly typed numeric like `int32` or `float64` with a far more permissible type parameter `T`. `T`, in this instance, is any type which fulfills the `Ordered` constraint (which we'll circle back to constraints shortly).

```go
func Max[T constraints.Ordered](x, y T) T {
    if x > y {
        return x
    }
    return y
}
```

When we call this function, we have to explicitly pass the type argument as part of the functions instantiation. Instatiation is a two part process where the compiler...

1.  substitutes the type argument for all instances of the respective type parameter. In our case, the two `T` arguments and one return value are swapped to be `int` specifically.
2.  checks that the two function arguments implement the constraints. The compiler will fail to instantiate if this step fails. Again, in our case, the compiler checks that 3 and 4 satisfy the `Ordered` constraint.

<!--listend-->

```go
max := Max[int](3,4)
```

It's also worth pointing our that the function call above both instantiates and runs the function. We could instantiate the function separately, which might be a slight optimisation in some cases.

```go
maxInt := Max[int]

max := maxInt(3,4)
```

As for data structures, these type parameters work the same way. Types can optionally have a type parameter list, and methods of that type must declare matching type lists in the receiver.

```go
type Grid[T any] struct {
    values        []T
    height, width int
}

func (g *Grid[T]) At(x, y int) T {
    return g.Values[(g.height * y) + x]
}

var grid Grid[int]
```

Notice the `any` keyword? It's now an alias for `interface{}`!


### Type sets and constraints {#type-sets-and-constraints}

So what are these "constraints" we've been tossing around?

Constraints are a new package in the standard library that describe type sets. Type sets are just lists of types which satisfy some target behavior. For example, the `Signed` constraint is the set of all signed integer types, and the `Integer` constraint is the union of `Signed` and `Unsigned`. To check if a type satisfies a constraint, the compiler just checks if that type is an element in the constraint's type set.

At the time of writing this, there are only six, simple constraints: `Signed`, `Unsigned`, `Integer`, `Float`, `Complex`, and `Ordered`. `Ordered` is the most permissive and includes all floats, integers, and strings; and was the constraint I reached for most often in initial testing.

```go
// Signed is a constraint that permits any signed integer type.
// If future releases of Go add new predeclared signed integer types,
// this constraint will be modified to include them.
type Signed interface {
        ~int | ~int8 | ~int16 | ~int32 | ~int64
}

// Integer is a constraint that permits any integer type.
// If future releases of Go add new predeclared integer types,
// this constraint will be modified to include them.
type Integer interface {
        Signed | Unsigned
}
```

You may have also noticed that these constraints are actually interfaces under the hood. Traditionally, interfaces have defined a 'method set' and every type which implements those methods implements that interface.

The other perspective, and one which is more relevant to generics, is that interfaces describe a set of _types_ and the method set is only a means by which we filter the set of _all types_ -- the empty interface. It seems only reasonable then that we should be able add a specific type to that list directly.

Well, as of `1.18beta1`, interfaces _can_ enumerate types directly by way of a type set (`Signed | Unsigned`, for example). Of course, method sets as we have known then are still 100% compatible and preferred in many cases.

In summary, type constraints are just interfaces and the types which satisfy those constraints are those enumerated by the interface. When you're defining a generic function with a constraint, you're basically defining a big list of all possible argument types.

For now, this flavor of type set interfaces can only be used as function constraints, but in the future I would like to see variables loosely typed according to a given constraint.


## Advent of Code {#advent-of-code}

Day 9 or "Smoke Basin" is a fun exercise in navigating grids which boils down to "can you find elements in a grid in which all surrounding 'neighbors' are larger than it". Before we dive into the puzzle logic, lets setup our data structures.

Fortunately, grids are common data structures in the Advent of Code, but unfortunately one that I've rewritten a number of times depending on the element type. My preferred approach is to structure the grid as a list and to define several helper methods to access elements with X,Y coordinates.

We'll need to directly compare Grid elements but would like this to be reused for, say, ASCII characters in the future, so the `Ordered` constraint makes the most sense.

```go
type Point struct {
        X, Y int
}

type Grid[T constraints.Ordered] struct {
        H, W int
        Vals []T
}
```

As for the helper methods, notice how the function receivers also specify the generic type `T`? That tells the compiler that these methods are applicable to any Grid which meets its constraint. A receiver like `(g *Grid[int])` would only be applicable to integer grids. Otherwise these are standard helper methods to access generic values in the grid, either by point coordinates, index, or relative direction.

```go
// Index returns the integer index value for a grid element given some point.
func (g *Grid[T]) Index(p *Point) int {
        return (p.Y * g.W) + p.X
}

// Point returns the a point object for a grid element given some index.
func (g *Grid[T]) Point(i int) *Point {
        return &Point{
                X: i % g.W,
                Y: i / g.W,
        }
}

// At returns the element found at some given point.
func (g *Grid[T]) At(p *Point) T {
        return g.Vals[g.Index(p)]
}

// InBounds returns true if the point is within the grid, and false if not.
func (g *Grid[T]) InBounds(p *Point) bool {
        return p.X >= 0 && p.X < g.W &&
                p.Y >= 0 && p.Y < g.H
}

// Neighbors returns a list of point objects for each (in-bound) element of the
// grid, given a list of directions. For example, the direction (1,0) would be
// the point to the right.
func (g *Grid[T]) Neighbors(p *Point, directions []*Point) (points []*Point) {
        for _, direction := range directions {
                tmp := p.Add(direction)
                if g.InBounds(tmp) {
                        points = append(points, tmp)
                }
        }
        return
}
```

Finally, the puzzle logic.

The puzzle input for day 9 was a grid of integers where each point represented the depth of the sea floor with 0 being the lowest and 9 being the highest. The first part of the puzzle is to find all of the low points (a point where the neighboring values are all greater) and add their values.

A simple solution is to iterate over the grid, check if each point is a "low point", and add the low point's values to a running total. There are a number of optimizations we could make here, but lets stick with the direct approach first.

```go
// IsLowPoint returns true if the given Point is lower than all its neighbors,
// and false if not.
func IsLowPoint[T constraints.Ordered](grid Grid[T], target *Point) bool {
        for _, p := range grid.Neighbors(point, FOUR_AXIS_DIRECTIONS) {
                if grid.At(p) <= grid.At(target) {
                        return false
                }
        }
        return true
}

// LowPoints returns a list of Points which correspond to all the low points in
// some given grid.
func LowPoints[T constraints.Ordered](grid Grid[T]) (points []*Point) {
        for i := range grid.Vals {
                point := grid.Point(i)
                if IsLowPoint(grid, point) {
                        points = append(points, point)
                }
        }
        return
}

// PartOne returns the sum of the values of all the low points in some given
// grid.
func PartOne(grid Grid[int]) (sum int) {
        for _, point := range LowPoints(grid) {
                sum += grid.At(point) + 1
        }
        return
}
```

A few things to note. In `PartOne`, we're actually specifying that our generic grid is a grid of integers. Although the addition operator is technically defined on strings for concatenation, the compiler knows that the return value must be an integer and the `Ordered` type set includes strings and floats. So to guarantee type safely, the compiler will enforce a strongly typed grid. The `LowPoints` and `IsLowPoint` functions only ever perform comparisons on grid values, so those can stay generic.

Part two is an iteration on the Grid we've just written, so I'll leave that as an exercise for you.


## Final thoughts {#final-thoughts}

Up until `1.18beta1`, I was frequently copying and pasting data structures and helper methods. In the best case, that led to code duplication. In the worst case, that led to unnecessary extraction and  abstraction. Generics feel like a handy way to inject flexibility into your code without resorting to re-use or adapter patterns, for example. That said, I'll have to see sufficiently complex implementations to form any lasting opinions.

At this point, I worry that -- like any new, shiny tool -- developers will look to cram generics wherever they can. Frankly, I think that generics will make the biggest impact in standard libraries -- not your application backend. The most obvious example is `math`, where currently _every_ function takes a `float64` and requires a significant amount of casting if you're working with integers (`int(math.Abs(float64(value)))`).

As for AOC, I'm all for using [container/heap](https://pkg.go.dev/container/heap) to implement a priority queue once in a while, but rewriting methods like `Abs`, `Max`, and `Min` is slow and inefficient. Even the standard 2-dimensional grid gets repetitive after a while. As a result, puzzlers have written their own [libraries of helper methods](https://github.com/Bogdanp/awesome-advent-of-code#go) to speed things along; contents range from simple data structures to stdin readers tailored to AOCs input.

I tried writing a library myself last year, but it felt brittle. Grids wont always contain integers and I should be able to compare strings just as easy as numerics. Interfaces might have been an option, but felt clumsy for my use case.

Enter: generics. I'm taking another stab with the help of `1.18beta1` -- all contributions are welcome.
