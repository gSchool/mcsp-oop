# Getters and Setters

As we saw at the end of the last section, there are times we want to hide internal data from being accessed _or_ mutated from outside the object (e.g. the `powerUps` array). But what do we do when we want a property to be accessible but not mutable? This is where getters come in. A getter is special type of function defined on an object which allows read-only access to a property. Additionally, they allow you to be alerted when a property is accessed, in case some other type of work needs to be done in that case. Here's a simple example of a getter at work.

```js
let myName = "Jay";
const obj = {
  get name() {
    return myName;
  },
};

console.log(obj.name); // Jay
obj.name = "Marcus";
console.log(obj.name); // Jay
```

A getter is defined with the `get` keyword and although it looks like a normal method, it is not invoked through the usual `()` syntax. It is actually invoked when you access a property normally with the dot syntax, as can be seen in the above example.

On the flip side, if you want to allow a property to be mutable, you can use a setter. Here's an example of that:

```js
let myName = "Jay";
const obj = {
  get name() {
    return myName;
  },
  set name(name) {
    myName = name;
  },
};

console.log(obj.name); // Jay
obj.name = "Anjali";
console.log(obj.name); // Anjali
```

Now you may be asking yourself how a getter/setter pair is better than just defining a normal property. If you allow access and mutation of a property, why not just expose it on the object? Well, one added benefit of a getter/setter pair is that we have more control over how properties are set and accessed. What if we wanted to keep track of every time a property is accessed or perform validation before accepting a property mutation? With a normal property, there's no way for us to hook into access/mutation, but with a getter/setter pair, we can just add some additional logic to our getter/setter functions like so:

```js
let myName = "Jay";
let timesAccessed = 0;
const obj = {
  get name() {
    timesAccessed++;
    return myName;
  },
  set name(name) {
    if (typeof name !== "string") {
      throw new Error("Invalid name given");
    }
    myName = name;
  },
};

obj.name;
obj.name;
console.log(timesAccessed); // 2
obj.name = 4; // Error: Invalid name given
```

Now that we have a basic understanding of getters and setters, let's return to the example we ended with in the previous section.

```js
const Player = (name, hp) => {
  const powerUps = [];

  return {
    name,
    hp,
    getStarPowerUp() {
      powerUps.push("star");
    },
    takeDamage() {
      if (!powerUps.includes("star")) {
        this.hp -= 1;
      }
    },
  };
};
```

We've already hidden some information by moving the `powerUps` property to a closure variable, but what about the `name` and `age` properties? Should we hide those as well?

The answer is...it depends. Part of good object-oriented design is separating an interface (those parts externally accessable) from its implementation (those parts internal to the object). The `name` property should certainly be readable from the outside, as we're not doing anything else with it, but probably not writable.

As for the `hp` property, do you want users of your object to be able to read the `hp` property directly, or just alert them when `hp` goes to zero? Although there's no clear answer to that question (it depends on the application) one thing that's certain is we that don't want to give arbitrary write access to this property and instead only allow it to be updated in pre-defined ways (e.g. `takeDamage` or `getMushroom`).

For our example, let's say that we don't want to expose the `hp` property at all, and only expose the read-only `name` property. To do the latter, we can make use of a getter.

```js
const Player = (name, hp) => {
  const powerUps = [];

  return {
    get name() {
      return name;
    },
    getStarPowerUp() {
      powerUps.push("star");
    },
    takeDamage() {
      if (!powerUps.includes("star")) {
        hp -= 1;
      }
    },
  };
};

const player = Player("Mario", 2);
player.name = "Luigi";
console.log(player.name); // Mario
```

Great! We've hidden all the internal details and only exposed the parts needed by the outside. As a side-benefit, we've also eliminated any use of `this` which can be confusing for beginners, thus making our code more readable. Our interface now consists of three things:

- A `name` property (read-only)
- A `getStarPowerUp` method
- A `takeDamage` method

If more properties/methods are needed later on, or we decide to expose an existing property, we can do that, but it's best to keep your interfaces to the bare minimum as it reduces the amount we expose to the outside, and thus the amount we're responsible for maintaining. In a small codebase, exposing extra information may not be such a big deal, but as your project grows, the objects you create may be used in hundreds of different places, and if those places relied on mutating the `powerUps` property, for example, and you change it's implementation later on, you'll break those usages.

## Object.freeze

An alternative method of achieving information hiding is to use `Object.freeze`. `Object.freeze` makes an object immutable, meaning that new properties cannot be added to or removed from it and the values of existing properties cannot be changed.

```js
const Player = (name, hp) => {
  const powerUps = [];

  return Object.freeze({
    name,
    getStarPowerUp() {
      powerUps.push("star");
    },
    takeDamage() {
      if (!powerUps.includes("star")) {
        hp -= 1;
      }
    },
  });
};

const player = Player("Mario", 2);
player.name = null; // No effect
delete player.name; // No effect
player.newProp = 4; // No effect
console.log(player.name); // Mario
console.log(player.newProp); // undefined
```

### !challenge

* type: code-snippet
* id: d77c0c85-af6e-42f3-a0cb-64e7b44d99df
* language: javascript
* title: Information Hiding

### !question

Let's return to our `Printer` factory function challenge from a previous section and refactor it to achieve information hiding. Here were the original specifications:

Name: `Printer`
Properties (Should be configurable via parameters):
- `name` - `String` (the name of the printer, e.g. 'Canon WiFi.')
- `sheetCount` - `Number` (The number of sheets in the printer. Defaults to zero if not provided.)
Methods:
- `addSheets` - Takes a number of pages as a parameter and adds that number to the `sheetCount` property.
- `printJob` - Takes two parameters (name: `String`, size: `Number`) as parameters, and prints a message to the console for each page in the job with the following format: `Printing [jobName] page [currentPage] of [pageCount]` and subtracts that number from the `paperCount` property. If the number of pages in the job exceeds `sheetCount`, throw an `Error` with a message of `Job failed: please refill paper tray!`.

Additionally, we're going to add two more constraints:

1. The `name` property should be read-only.
1. The `sheetCount` property should be read-only.

### !end-question

### !placeholder

```js
const Printer = (name, sheetCount = 0) => {
  return {
    name,
    sheetCount,
    addSheets(numSheets) {
      this.sheetCount += numSheets;
    },
    printJob(name, size) {
      if(size > this.sheetCount) throw new Error("Job failed: please refill paper tray!");

      for(let i = 1; i <= size; i++) {
        console.log(`Printing ${name} page ${i} of ${size}`);
      }
      this.sheetCount -= size;
    }
  }
};
```

### !end-placeholder

### !tests

```js
describe('Printer', () => {
  it('`name` property is configurable via parameters', () => {
    const subject = Printer("Canon Wifi");
    expect(subject.name).to.equal("Canon Wifi");
  });

  it('`sheetCount` property is configurable via parameters', () => {
    const subject = Printer("Canon Wifi", 45);
    expect(subject.sheetCount).to.equal(45);
  });

  it('defaults `sheetCount` property to 0 if none provided', () => {
    const subject = Printer("Canon Wifi");
    expect(subject.sheetCount).to.equal(0);
  });

  it('`.addSheets` method updates `sheetCount`', () => {
    const subject = Printer("Canon Wifi", 20);
    subject.addSheets(15);
    expect(subject.sheetCount).to.equal(35);
  });

  it('`.printJob` method prints a message for each page in the job ', () => {
    const subject = Printer("Canon Wifi", 5);
    const logStub = sinon.stub(console, "log").callsFake(() => {});

    subject.printJob('Essay.docx', 3);

    const logCalls = logStub.getCalls();
    expect(logCalls[0].args[0]).to.equal('Printing Essay.docx page 1 of 3');
    expect(logCalls[1].args[0]).to.equal('Printing Essay.docx page 2 of 3');
    expect(logCalls[2].args[0]).to.equal('Printing Essay.docx page 3 of 3');
    logStub.restore();
  });

  it('`.printJob` method subtracts the job size from `sheetCount` ', () => {
    const subject = Printer("Canon Wifi", 50);
    const logStub = sinon.stub(console, "log").callsFake(() => {});

    subject.printJob('Term Paper.docx', 25);
    expect(subject.sheetCount).to.equal(25);

    logStub.restore();
  });

  it('`.printJob` method throws an error if the job size exceeds the `sheetCount`', () => {
    const subject = Printer("Canon Wifi", 4);
    const logStub = sinon.stub(console, "log").callsFake(() => {});

    expect(() => subject.printJob('Court Proceedings.pdf', 10)).to.throw('Job failed: please refill paper tray!');

    logStub.restore();
  });

  it('`sheetCount` property is read-only', () => {
    const subject = Printer("Canon Wifi", 4);
    expect(() => subject.sheetCount = 20).to.throw(TypeError);
    expect(subject.sheetCount).to.equal(4);
  });

  it('`name` property is read-only', () => {
    const subject = Printer("Canon Wifi", 4);
    expect(() => subject.name = "Epson Wifi").to.throw(TypeError);
    expect(subject.name).to.equal("Canon Wifi");
  });
});
```

### !end-tests

### !end-challenge