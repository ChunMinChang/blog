---
layout: post
title: Why Is Overengineering Bad
date: 2020-10-25 07:44 +0800
category: [Programming]
tags: [Common programming model]
comments: true
---

The answer is it usually causes confusions and deteriorate the code readability.

The ambitious programmers sometimes are unawareness of being over-engineering their code. This post proposes a simple way to check if the code is overengineered, and the steps to avoid it.

<!--read more-->

## What Is Overengineering

My definition is

> embedding personal assumptions in the code

The common symptoms are:

- The code is solving a problem that doesn't exist
- The code does way more than what the necessary requirements need based on personal speculation
- The code optimizes for the conditions that have never been true
- The code doesn't solve any real problem

An example is to build a giant gold mining factory in a small town only because some alluvial gold are found in a river near the town. The factory builder/owner *believe*s there are some gold mines near the town but there is no any sign showing there are mineral rich fields around, except for the onwser's crazy believe.

Anyone who has no idea about the town and the actual working status of the factory will assume the town produces a lots of gold when they see that giant gold mining factory. A new city mayer may plan to build a railway to transport the gold mined from there to city. However, the real yeild of the gold is less than the storage volume of a track. If the city mayer infers the gold yeild from the size of the factory, instead of checking the real yeild reports, the city will waste tons of money.

In real life, the overengineering result often delivers misleading messages and lead the observers to make wrong assumptions. The developing, maintaining, and time effort wasted on meeting the unnecessary requirements can be expensive. In short, overengineering can bring less benefits than the damages it causes.

### Example

Here is an example. The goal is to build a web-page with a button that will print a log when the buttons is clicked. It's pretty simple.

#### Requirements

- A web page with a button *B*
- *B* will print a log in the console, via `console.log()`, when it's clicked

#### Overengineered Code

```html
<!-- button.html -->
<html>
  <head>
    <title>demo</title>
    <script src="button.js"></script>
  </head>
  <body>
    <button id="btn">uninit</button>
  </body>
</html>
```

```javascript
/* button.js */
window.addEventListener("DOMContentLoaded", (event) => {
  Init();
  AddEventListeners();
});

function Init() {
  let btn = document.getElementById("btn");
  btn.innerText = BUTTON_TEXT["inited"];
}

function AddEventListeners() {
  let btn = document.getElementById("btn");
  for (const [key, value] of Object.entries(EVENTS)) {
    console.log(`Add listener: ${value} for ${key}`);
    btn.addEventListener(key, value);
  }
}

const EVENTS = {
  "click": (event) => {
    console.log("on"+ event.type);
  },
};

const BUTTON_TEXT = {
  "inited": "come on",
};

```
(live demo [here](https://jsfiddle.net/y5r7ovgq/))

Before I judge whether the above code is overengineered. What do you feel about above code? If you don't know what the requirements are, what requirements you will guess the code needs to fulfill?

The following is my guess, if I didn't know what the requirements are:

- A web page with a button *B*
- *B* will print a log in the console when it's clicked
- button *B* will listen *more* events in the future

The thrid assumption is made by reading `AddEventListeners` and `EVENTS` stuff. The above code leaves a **room** to **add more listeners**! I'll guess there are some follow-up patches to add more event-listeners.

The code auther may "assume" the code will be changed to something as follows some day in the "future":

```javascript
/* button.js */
window.addEventListener("DOMContentLoaded", (event) => {
  Init();
  AddEventListeners();
});

function Init() {
  let btn = document.getElementById("btn");
  btn.innerText = BUTTON_TEXT["unhovered"];
  // Other initialization like style properties ...
}

function AddEventListeners() {
  let btn = document.getElementById("btn");
  for (const [key, value] of Object.entries(EVENTS)) {
    console.log(`Add listener: ${value} for ${key}`);
    btn.addEventListener(key, value);
  }
}

const EVENTS = {
  "mouseover": (event) => {
    console.log("on"+ event.type);
    event.target.innerText = BUTTON_TEXT["hovered"];
    // do something else ...
  },
  "mouseout": (event) => {
    console.log("on"+ event.type);
    event.target.innerText = BUTTON_TEXT["unhovered"];
    // do something else ...
  },
  "mousedown": (event) => {
    console.log("on"+ event.type);
    event.target.innerText = BUTTON_TEXT["clicked"];
    // do something else ...
  },
  "mouseup": (event) => {
    console.log("on"+ event.type);
    event.target.innerText = BUTTON_TEXT["unhovered"];
    // do something else ...
  },
  "dblclick": (event) => {
    console.log("on"+ event.type);
    event.target.innerText = BUTTON_TEXT["dbclicked"];
    // do something else ...
  },
};

const BUTTON_TEXT = {
  "unhovered": "come on",
  "hovered": "try me",
  "clicked": "you really did!",
  "dbclicked": "itchy!",
};
```
(live demo [here](https://jsfiddle.net/kwLc3s6f/))

In that case, the "left-room" makes sense. There will be some follow-up patches come.

However, if there is no plan to do so, then the third guess will never be true. There is no any benefit to leave the "room" in advance, without any plan and actual needs. It only causes confusions to the reader.

Here is what happens when reading a code that does "more" than what the requirements are. The above code indeed fulfills the requirements. But the "doing-more" part will lead the reader to make a wrong assumption. The code author may think it's better to leave a room, "in case" it needs to listen more events. But it may cause trouble when someone wants to refactor the code since they "assume" their new design need to accommodate "multiple" event-listeners.

In my experience, most of the time, other programmers who need to refactor the code will notice the "left-room" is redundant, after reading more related code in other connected modules. At the end, they will delete the "more"/"left-room" part. This situation is less ideal since they need to spend time and efforts to figure out what are redundant and what parts need to be removed.

#### Proper Code

It's better to write a simple-enough code that can meets the requirements at the beginning. That is, we should do this to achieve what we want:

```html
<!-- button.html -->
<html>
  <head>
    <title>demo</title>
    <script>
      window.addEventListener("DOMContentLoaded", (event) => {
        let btn = document.getElementById("btn");
        btn.onclick = (event) => {
          console.log("on"+ event.type);
        };
      });
    </script>
  </head>
  <body>
    <button id="btn">come on</button>
  </body>
</html>
```
(live demo [here](https://jsfiddle.net/rdy07s1L/))

## How to Avoid Overengineering

While the overengineering part can be minimized when the code is being reviewed, the best approach is to avoid it in the first place. Here are steps to avoid overengineering:

1. List the actual requirements
2. Compare what you did with the requirements above
3. Remove the unnecessary parts serving for the unplanned "future" needs
   - Can this variable/class/function/interface/module be merged/omitted?
4. Add comments to those necessary "doing more" parts to avoid confusions
5. Ask review from other programmers you trust

## Let's Wrap Up

The key to avoid overengineering is to refrain yourselves from adding more code that prepares for the unnecessary "future" needs. Most the time they are just looks-good-to-auther-only code. If there is no "planned" future needs, don't add code based on your personal assumption. I've seen a piece of code commented with `/* This is used to test XXX for a couple of weeks. */` but yet it lives for a couple of "years" instead. The future can be unpredictable, and so does your time management and memory.

I know sometimes it's hard. I've been there, when I am eager to show off my skills. But, in fact, it always takes more time to make it pass the code-review by doing so. It usually takes lots of time to explain the confusions it brings. It's better to focus on what the actual needs instead of the unplanned "future" needs. It's better to *trust* your teammates can write good code to meet the requirements , in the future, *without* the helps or hints from the "doing more" parts you left. You need to trust your teammates, really.

Don't build an electric factory when you only need a plugin. 99 percent of the time, *less is more*.
