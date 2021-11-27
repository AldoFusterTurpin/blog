---
title: "Paddock Management"
date: 2021-08-22T20:32:33+02:00
draft: false
---
Today I want to share with you my solution to a code interview problem that I came across some time ago. Even the problem is not too complex, it includes a set of challenges that let me show you the beauty of Go and how we can apply best practices to make our code more readable, understandable and testeable. 

This blog Post is NOT an introduction to the Go language. Nevertheless, if you are starting with Go (you know at least the basic syntax), stick arround and I will be happy to share with you some useful principles. 

Even if your are an experienced engenieer you can compare how you tackle the problem against my solution and even think different strategies to solve the problem. If you want, you can tell me what you think about it or what would you do different.

In this blog post we will take a look at some cool concepts:
- How Test Driven Development (TDD) can help us tackle problems from the beginning.
- How to marshal and unmarshal (akka manipulate) JSON files in Go.
- How to separate the domain logic of the application from the representation of the data to make sure the use cases are reusable no matter the data format.

You can find the full source code of the solution [here](https://github.com/AldoFusterTurpin/coding-challenges/tree/master/paddock_management). 

Qucik note about the repo (for the curious): I have a repo called [coding-challenges](https://github.com/AldoFusterTurpin/coding-challenges) where I have more coding challenges a part from that. In that repo, there is a folder called paddock_management that contains the full source code of the solution (the one I shared with you in the previous sentence). 

In a production or enterprise environment, we will have an independent git repo with the code because this go project is a Golang module. In this particular case I just created the module here because I want to group my challenges in one repo. 

[If you did not understood anything, don't worry, it's also difficult for me to understand myself from time to time 😂]

Let's get started! 

## Problem statement
The  INPUT of the problem is composed from two JSON files (tip: open them in new tabs/windows):

- [paddockTypes.json](https://github.com/AldoFusterTurpin/coding-challenges/blob/master/paddock_management/input/paddockTypes.json): represents the definition of some paddock types (with an id and a name).

- [paddocks.json](https://github.com/AldoFusterTurpin/coding-challenges/blob/master/paddock_management/input/paddocks.json): this file contains an array of paddocks. Each element of the array contains some fields that are not relevant for the problem. What is relevant is the fact that the attribute paddockTypeId references the corresponding paddock type found in paddockTypes.json.

What do we need to do in order to solve the problem?

Return an array with the paddockTypes, ordered in decreasing order by the TOTAL sum of the number of hectares planted for each of them, found in the paddocks arrangement.

## Problem solution summary

What would be the expected output with paddocks.json and paddockTypes.json as input?

result.json (expected output would be): 

```json
[
  {
    "id": 2,
    "name": "AVELLANOS",
    "HectaresSum": 88008
  },
  {
    "id": 1,
    "name": "PALTOS",
    "HectaresSum": 56259
  },
  {
    "id": 3,
    "name": "CEREZAS",
    "HectaresSum": 38073
  },
  {
    "id": 4,
    "name": "NOGALES",
    "HectaresSum": 2151
  }
]
```

To obtain this JSON, we need to iterate over paddocks.json and use the paddockManagerId attribute as it indicates to which paddockTpe is related to.

For example, in the expected result, the first element of the JSON has an HectaresSum of 88008.

```json
  {
    "id": 2,
    "name": "AVELLANOS",
    "HectaresSum": 88008
  }
```

 How did we obtain that number? If we take all the paddock elements in paddocks.js and filter just the ones with "id": 2 and sum their hectares, we will obtain 88008, the "HectaresSum".

 Conceptually we have to group all the paddocks in paddocks.json that have the same paddockTypeId and sum the area for each group. Each of those groups will be an element of the array found in result.json. Finally, we have to sort by HectaresSum (decreasing order) all the objects in the array.

 ### A quick introduction to TDD
 [TDD](https://martinfowler.com/bliki/TestDrivenDevelopment.html) stands for Test Driven Development and is a technique to create software. Even some people say it is a good methodlogy to cover your software with test, the truth is that TDD is a great technique to DESIGN software in a way that it is more readable, mantainable and scalable. So even having more test coverage can be a consequence of applying TDD, the truth is that TDD focuses in writing better software.

 What TDD promotes is the use of three little steps in order to make progress when programming:
 - First of all create a RED test that expresses what do you want to achieve giving some input (RED phase).
 - Second, create the minimmum amount of (production) code to make the tets pass (GREEN phase).
 - Third, refactor your existing code in a way that the test is still green but your production code is cleaner.(REFACTOR phase). Even it can be tempting to include some features to production code when refactoring, don't do that because it can introduce bugs without noticing it.
 - Repeat above cycle until you have finished the whole system. 

![TDD image](/tdd.jpg)

 TDD is a huge topic today and I encourage you to go deeper. This is a super quick intro to understand how we are proceding.

 Some references:
 - [Learn Go with tests](https://quii.gitbook.io/learn-go-with-tests/)
 - [Unit testing made easy in Go](https://medium.com/rungo/unit-testing-made-easy-in-go-25077669318)
 - [Thoughts on TDD](https://dev.to/toureholder/the-really-effective-part-of-tdd-is-not-so-much-whether-you-write-the-test-first-according-to-uncle-bob-3h6n)

 ## Quick note for TDD gurus
Even though we take most of TDD when we apply the 3-steps cycle (Red, Green, Refactor) multiple times iterating over the problem and add a lot of test cases (inputs), we can  also see the benefits of TDD when the cycle is repeted few times: we will see it now. 

 ## Let's solve the problem in Go.
 First of all, we notice that the input of our problem is composed by two JSON files. Nevertheless, we want to solve the problem in terms of the domain, no matter the format of the data.

 That's why the first thing we need to do is create a Test with the input and the expected output. It's also a good idea map the JSON fields into Go structs as we will be manipulating domain objects, not JSON fields. That's exactly what we are doing in the first commit. Y
 
 You can check it out with 
 ```
 git checkout c8af77b27cca522d67f1e22fa8f71e03a1ddc922
 ```

 We first create a file [main_test.go](https://github.com/AldoFusterTurpin/coding-challenges/commit/c8af77b27cca522d67f1e22fa8f71e03a1ddc922). This file includes the definition of the structs types that represents the JSONs previously explained. It also includes the expected result in terms of the domain. 

 What is really important here is that we are expressing the problem data in terms of "objects" (structs but OOP concepts apply) in the domain space, nothing related to JSON. Is it true that we are indicating which field of the structs map to which JSON attribute, but the whole problem will be solved in terms of the domain, no JSON language, as we will see.

```Go
package main

import (
	"reflect"
	"testing"
)

type PaddockType struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

type Paddock struct {
	PaddockManagerID int `json:"paddockManagerId"`
	FarmID           int `json:"farmId"`
	PaddockTypeID    int `json:"paddockTypeId"`
	HarvestYear      int `json:"harvestYear"`
	Area             int `json:"area"`
}

type ResultType struct {
	PaddockType
	HectaresSum int
}

func TestPaddockManager(t *testing.T) {
	tt := map[string]struct {
		paddockTypes []PaddockType
		paddocks     []Paddock
		expected     []ResultType
	}{
		"simple_input": {
			paddockTypes: []PaddockType{
				PaddockType{ID: 1, Name: "PALTOS"},
				PaddockType{ID: 2, Name: "AVELLANOS"},
				PaddockType{ID: 3, Name: "CEREZAS"},
				PaddockType{ID: 4, Name: "NOGALES"},
			},
			paddocks: []Paddock{
				Paddock{PaddockManagerID: 6, FarmID: 1, PaddockTypeID: 1, HarvestYear: 2019, Area: 1200},
				Paddock{PaddockManagerID: 1, FarmID: 3, PaddockTypeID: 4, HarvestYear: 2019, Area: 500},
				Paddock{PaddockManagerID: 5, FarmID: 3, PaddockTypeID: 2, HarvestYear: 2020, Area: 20000},
				Paddock{PaddockManagerID: 2, FarmID: 2, PaddockTypeID: 3, HarvestYear: 2021, Area: 8401},
				Paddock{PaddockManagerID: 3, FarmID: 1, PaddockTypeID: 1, HarvestYear: 2020, Area: 2877},
				Paddock{PaddockManagerID: 5, FarmID: 2, PaddockTypeID: 2, HarvestYear: 2017, Area: 15902},
				Paddock{PaddockManagerID: 3, FarmID: 3, PaddockTypeID: 2, HarvestYear: 2018, Area: 1736},
				Paddock{PaddockManagerID: 2, FarmID: 3, PaddockTypeID: 3, HarvestYear: 2020, Area: 2965},
				Paddock{PaddockManagerID: 4, FarmID: 3, PaddockTypeID: 4, HarvestYear: 2018, Area: 1651},
				Paddock{PaddockManagerID: 5, FarmID: 1, PaddockTypeID: 1, HarvestYear: 2018, Area: 700},
				Paddock{PaddockManagerID: 1, FarmID: 2, PaddockTypeID: 1, HarvestYear: 2019, Area: 7956},
				Paddock{PaddockManagerID: 5, FarmID: 3, PaddockTypeID: 2, HarvestYear: 2020, Area: 3745},
				Paddock{PaddockManagerID: 6, FarmID: 1, PaddockTypeID: 3, HarvestYear: 2021, Area: 11362},
				Paddock{PaddockManagerID: 2, FarmID: 3, PaddockTypeID: 3, HarvestYear: 2021, Area: 300},
				Paddock{PaddockManagerID: 3, FarmID: 2, PaddockTypeID: 2, HarvestYear: 2020, Area: 19188},
				Paddock{PaddockManagerID: 3, FarmID: 1, PaddockTypeID: 1, HarvestYear: 2019, Area: 17137},
				Paddock{PaddockManagerID: 4, FarmID: 3, PaddockTypeID: 2, HarvestYear: 2020, Area: 100},
				Paddock{PaddockManagerID: 2, FarmID: 1, PaddockTypeID: 3, HarvestYear: 2019, Area: 11845},
				Paddock{PaddockManagerID: 5, FarmID: 2, PaddockTypeID: 1, HarvestYear: 2018, Area: 15969},
				Paddock{PaddockManagerID: 1, FarmID: 3, PaddockTypeID: 1, HarvestYear: 2029, Area: 10420},
				Paddock{PaddockManagerID: 5, FarmID: 2, PaddockTypeID: 3, HarvestYear: 2010, Area: 3200},
				Paddock{PaddockManagerID: 6, FarmID: 1, PaddockTypeID: 2, HarvestYear: 2012, Area: 10587},
				Paddock{PaddockManagerID: 2, FarmID: 2, PaddockTypeID: 2, HarvestYear: 2018, Area: 16750},
			},
			expected: []ResultType{
				ResultType{
					PaddockType: PaddockType{
						ID:   2,
						Name: "AVELLANOS",
					},
					HectaresSum: 88008,
				},
				ResultType{
					PaddockType: PaddockType{
						ID:   1,
						Name: "PALTOS",
					},
					HectaresSum: 56259,
				},
				ResultType{
					PaddockType: PaddockType{
						ID:   3,
						Name: "CEREZAS",
					},
					HectaresSum: 38073,
				},
				ResultType{
					PaddockType: PaddockType{
						ID:   4,
						Name: "NOGALES",
					},
					HectaresSum: 2151,
				},
			},
		},
	}

	for name, tc := range tt {
		t.Run(name, func(t *testing.T) {
			got := GetResult(tc.paddockTypes, tc.paddocks)
			if !reflect.DeepEqual(tc.expected, got) {
				t.Errorf("Expected %v but got: %v.", tc.expected, got)
			}
		})
	}
}
```

First of all, we define the structs that represent the JSON files. After that, we crate a test using what is called [Table Driven Test](https://dave.cheney.net/2013/06/09/writing-table-driven-tests-in-go). It let us specify a struct with all the inputs for the test with the corresponding expected output. 