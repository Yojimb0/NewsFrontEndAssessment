NewsFrontEndAssessment
======================

Technical assessment for a front-end job opportunity at News (Australia)



Question 1
---
```
(function() {
	var index,
		length = 10;
	for (index = 0; index < length; index++) {
		setTimeout(function(specificIndex) {
			console.log(specificIndex);
		}(index), 100);
	}
})();
```
First issue, `this.length`: in this context `this` refers to the `Window Object`, which have a length property (So this doesn't raise any error), but its value is 0. What we wanted to have here, was the variable `length` declared just above.
Second issue, all the `console.log` are displaying `10`. Because with the `setTimeout` function, we created 10 closures that share the same `index` value. We need to give it a specific value of `index` in each iteration of the loop. A solution with a good browser compatibility would be to force a parameter to the `setTimeout` callback, but we could have use the ES5 function `bind` too.
Question 2---
```
window.onload = function() {
	document.getElementById('test').innerHTML = 'Hello World';
};```

To get the code be run after the DOM has loaded, we simply set the `window.onload` event, to execute our code.

In the `question-2.html` file, I've put my `<script>` tag before the `<div id="test"></div>` for demonstration purpose, but it would obviously be better to have it the lowest possible in the file.


Question 3
---
```
(function() {
	if(navigator.appVersion.indexOf("MSIE 7.")!=-1){
		alert('Hello World');
	}
})();
```

Pure JS: Basically looking for the IE7 specific string in the UserAgent, stored in the `navigator` object.

But I would prefer to have some HTML conditional comments to load specific files separately, to keep clean files for the most used browser, and fit with a progressive enhancement approach (or here, more a graceful degradation approach)

Working on a mac, and doesn't have a browser testing tool right now, so I didn't created a file for this answer


Question 4
---
```
function hello(name, age) {
	var msg = name + ", who is " + age + " years old, says hi!";
	
	return function sayHello() {
		console.log(msg);
	};
}

var sayHelloToJohn = hello("John", 33);

sayHelloToJohn();
```

This way, we still say hello to John, but all the state is nice and tidy, hidden in the local scope of our hello() function.



Question 5
---
```
(function() {
	var array = [
			function(callback) {
				console.log('first function called in ' + (new Date().getTime() - timestamp) + 'ms');
				callback();
			},
			function(callback) {
				console.log('second function called in ' + (new Date().getTime() - timestamp) + 'ms');
				callback();
			},
			function(callback) {
				console.log('third function called in ' + (new Date().getTime() - timestamp) + 'ms');
				callback();
			}
		],
		timestamp = new Date().getTime();

	function flow(array, callback) {

		array[0]( function(){
			array[1]( function(){
				array[2]( function(){
					callback();
				});
			});
		});

	};

	flow(array, function() {
		console.log('all functions finished in ' + (new Date().getTime() - timestamp) + 'ms');
	});

})();
```

Here we want to execute each function when its predecessor is over, so we simply give each function the next one as the callback parameter. We finish the "callback-hell" by calling the `flow`'s callback, which gonna display the final message.

We could have use Promises to reduce the callback nesting.


Question 6
---

```
(function() {
	function makeMyArrayUpperCaseItsUselessButHeyItWorks(array){
		var l = array.length;
		for(var i=0; i<l; i++){
			array[i] = array[i].toUpperCase();
		}
		return array;
	}
	
	function isArray(array, callback) {
		var returnValue = typeof(array) === 'object' && (array instanceof Array);

		// If falsy, we return it right away.
		if(!returnValue) return returnValue;

		// Is the callback a function ?
		if(typeof callback === 'function'){
			return callback(array);
		}else{
			return returnValue;
		}
		
	}
	
	console.log(isArray('a'));
	console.log(isArray('a', makeMyArrayUpperCaseItsUselessButHeyItWorks));
	console.log(isArray(['item1','item2','item3']));
	console.log(isArray(['item1','item2','item3'], makeMyArrayUpperCaseItsUselessButHeyItWorks));
	
})();

```

For this question, I'm not sure I've understood the instruction very well. I was ask to modify the code so that the return value can also be returned with a callback function.

So here my understanding, the isArray function can take a function as a second parameter, which will be executed as a callback if the first parameter is proved to be an array. So I chose a random action to apply to my array, just to demonstrate. Hope it was the thing to do.



Question 7
---

```
(function() {
	var html='',
		element,
		index,
		length,
		content = document.getElementById('content'),
		data = [
			{id: 1, name: 'John',   color: 'green'},
			{id: 2, name: 'Sally',  color: 'pink'},
			{id: 3, name: 'Andrew', color: 'blue'},
			{id: 4, name: 'Katie',  color: 'purple'}
		];
	for (index = 0, length = data.length; index < length; index++) {
		html += '<li id="'+data[index].id+'" style="color:'+data[index].color+'">';
		html += '	<strong>' + data[index].name + '</strong>';
		html += '</li>';
	}

	content.innerHTML = html;


})();
```

Here, I'm asked to reduce the number of repaint and reflows. The basic task to do in this case is to reduce the amount of DOM manipulation, specifically in  loops (Where it can multiply easily).

So here I removed all the `createElement`, `setAttribute`, style modification, appending functions, to replace everything by a big string. This string is filled on every iteration of the loop with the element for each item of the data array

At the end of the loop, once the string is complete, I use just one DOM manipulation function : `innerHTML`, [which is quite efficient compared to other method](http://stackoverflow.com/questions/18393981/append-vs-html-vs-innerhtml-performance)

We can see the performance gain here : [Before](http://d.pr/i/FKU1) vs [After](http://d.pr/i/lp6k). We see that there's way less HTML Parsing. But this screenshots aren't very useful: I took them on Chrome, which is intelligent enough to wait for a bunch of repaint-actions before really repainting.

An other issue of this technique is that string concatenation is done very VERY poorly on IE7 and below, and for a bigger `data` object an `array.join` should have had better results.



Question 8
---

```
(function(window) {

	/* API
	---------------------------------------*/
	var document = window.document;
	
	// Wrapper constructor, we get the selector
	// We use it to retrieve the node list, thanks to document.querySelectorAll
	function myWrapper(selector){
		this.selector = selector;
		this.element = document.querySelectorAll(selector);
	}
	
	// Creating the main object containing our chain-able functions
	myWrapper.prototype = {
		show: function(){
			// We transform ou NodeList to an Array,
			// to be able to "forEach" it
			var array = Array.prototype.slice.call(this.element, 0);
			array.forEach(function(node) {
				node.style.display = 'block';
			});
			// Returning "this" allow us to pass the current state to the next function
			// Creating a chain-able pattern
			return this;
		},
		hide: function(){
			var array = Array.prototype.slice.call(this.element, 0);
			array.forEach(function(node) {
				node.style.display = 'none';
			});
			return this;
		},
		blue: function(){
			var array = Array.prototype.slice.call(this.element, 0);
			array.forEach(function(node) {
				node.style.color = 'blue';
			});
			return this;
		}
	}
	
	// We bind our wrapper to a shorthand function: $
	window.$ = function(selector){
		return new myWrapper(selector);
	}

	/* USAGE
	---------------------------------------*/
	$('.test').hide().show().blue();

})(window);
```

Comments in the code.


Question 9
---

```
(function() {
	var boxes = [
		document.getElementById("box1"),
		document.getElementById("box2"),
		document.getElementById("box3")
	];

	boxes[0].addEventListener("mousemove", function(event) {
		console.log("Box 1");
		event.stopPropagation();
	});

	boxes[1].addEventListener("mousemove", function(event) {
		console.log("Box 2");
		event.stopPropagation();
	});

	boxes[2].addEventListener("mousemove", function(event) {
		console.log("Box 3");
		event.stopPropagation();
	});

})();

```

The problem in this piece of code, was that too many event were fired. In Javascript event are bubbling, meaning that the event is gonna climb the DOM up to the root element "asking" each element if they have an eventListener for its type.

So Here, when we roll over the box3:

* \#box3 mousemove listener fires
* Then bubbling to his parent: #box2, which has a mousemove listener too, fires too
* Same for #box1


[Here a scheme of the bubbling process](http://d.pr/i/DEIX)

To solve this: `event.stopPropagation();` that let us stop the bubbling phase when we want, to avoid rising other listeners on the parent elements.

