# HBTree.js
Library for decision making using Hybrid Behaviour Trees, an evolution of classic Behaviour Trees, which adds a dregree of freedom to the user, allowing to mix flow graphs with Behaviour Trees. The idea is to be able to tweak and play with scenario and virtual characters properties before using them to determine the behaviour. 

# How to use HBTree.js

Every time you wnat to use HBTree.js, you must create a context, understanding by context the surroundings of the virtual character we want to evaluate. 

```javascript
var context = new HBTContext();
```

As this library aims to evaluate scene properties, as distances between virtual characters or some points of the scene, there are some nodes which requires methods from the render engine. To solve that and make this library usable for every Javascript render engine, we have developed an API, so if you are using a different render engine, you just have to overwrite the methods of the facade.js

```javascript
context.facade.getEntityPosition = function ( entity )
{
  //your code
}
```

After that, you should load a JSON file containing a hybrid behaviour (created on [Medusa](https://webglstudio.org/users/dmoreno/projects/ondev/saucemedusa/) or downloaded from the [open repository](https://webglstudio.org/users/dmoreno/projects/repo/))  

```javascript
context.current_graph.graph.configure(loaded_behaviour);
```

In case you want to load more than one behaviour, there is a container inside the context to store the different behaviours

```javascript
var new_HBTGraph = context.addHBTGraph(name);
```

To execute the Hybrid Behaviour Tree: 

```javascript
var behaviour = context.evaluate(entity, dt, graph); //the third parameter could be the current one or another
// do whatever you want with the info provided in behaviour
```

The behaviour variable contains the type of task which has been reached in the evaluation and the necessary information. For example, if it is a SimpleAnimate node, the necessary information will be the name of the animation to reproduce, the weight od the animation and the playback speed. 

As this library is still on development, new nodes will be added according to the necessities on evaluation stages, as well as modifications on the existing ones to improve it. 
