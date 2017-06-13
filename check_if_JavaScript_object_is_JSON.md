# How to check if JavaScript object is JSON


I have a nested JSON object that I need to loop through, and the value of each key could be a String, JSON array or another JSON object.
Depending on the type of object, I need to carry out different operations.
Is there any way I can check the type of the object to see if it is a String, JSON object or JSON array?

I tried using typeof and instanceof but both didn't seem to work, as typeof will return an object for both JSON object and array, 
and instanceof gives an error when I do obj instanceof JSON.

To be more specific, after parsing the JSON into a JS object, is there any way I can check if it is a normal string, 
or an object with keys and values (from a JSON object), or an array (from a JSON array)?


I'd check the constructor attribute.

e.g.

```js
var stringConstructor = "test".constructor;
var arrayConstructor = [].constructor;
var objectConstructor = {}.constructor;

function whatIsIt(object) {
    if (object === null) {
        return "null";
    }
    else if (object === undefined) {
        return "undefined";
    }
    else if (object.constructor === stringConstructor) {
        return "String";
    }
    else if (object.constructor === arrayConstructor) {
        return "Array";
    }
    else if (object.constructor === objectConstructor) {
        return "Object";
    }
    else {
        return "don't know";
    }
}

var testSubjects = ["string", [1,2,3], {foo: "bar"}, 4];

for (var i=0, len = testSubjects.length; i < len; i++) {
    alert(whatIsIt(testSubjects[i]));
}
```
