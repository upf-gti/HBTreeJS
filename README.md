# HBTree.js
Library for decision making using Hybrid Behaviour Trees, an evolution of classic Behaviour Trees, which adds a dregree of freedom to the user, allowing to mix flow graphs with Behaviour Trees. The idea is to be able to tweak and play with scenario and virtual characters properties before using them to determine the behaviour. 

# How to use HBTree.js

Every time you want to use HBTree.js, you must create a context, understanding by context the surroundings of the virtual character we want to evaluate, and a graph, which could be understood as the "brain" of the virtual character. 

```javascript
var context = new HBTContext();
var hbt_graph = new HBTGraph("graph_name");
```

As this library aims to evaluate scene properties, as distances between virtual characters or some points of the scene, there are some nodes which requires methods from the render engine. To solve that and make this library usable for every Javascript render engine, we have developed an API, so if you are using a different render engine, you just have to overwrite the methods of the Facade, class privided in the bottom of the HBTree.js file:

```javascript
context.facade.getEntityPosition = function ( entity )
{
  //your code
}
```

After that, you should load a JSON file containing a hybrid behaviour (created on [Medusa](https://webglstudio.org/users/dmoreno/projects/ondev/saucemedusa/) or downloaded from the [open repository](https://webglstudio.org/users/dmoreno/projects/repo/))  

```javascript
hbt_graph.graph.configure(loaded_behaviour); //loaded_behaviour must be an object from the parsed JSON, not the JSON
```

To execute the Hybrid Behaviour Tree: 

```javascript
var behaviour = hbt_graph.runBehaviour( entity, context ,dt );
// do whatever you want with the info provided in behaviour
```

The behaviour variable contains the type of task which has been reached in the evaluation and the necessary information. For example, if it is a SimpleAnimate node, the necessary information will be the name of the animation to reproduce, the weight od the animation and the playback speed. It is important to understand that this library does not execute the behaviours, as it provides the freedom to do whatever the developer desires with the information. 

As this library is still on development, new nodes will be added according to the necessities on evaluation stages, as well as modifications on the existing ones to improve it. 

# How to code new Nodes

```javascript
//node constructor class
function MyNode()
{
    this.shape = 2;
    this.color = "#1E1E1E"
    this.boxcolor = "#999";
    this.addInput("","path", {pos:[w*0.5,-LiteGraph.NODE_TITLE_HEIGHT], dir:LiteGraph.UP});
    this.addInput("value","number", {pos:[0,60], dir:LiteGraph.LEFT});
    this.addOutput("","path", {pos:[w*0.5,h], dir:LiteGraph.DOWN});
	  this.properties = {};
    this.horizontal = true;
	  this.widgets_up = true;
}

MyNode.prototype.onStart = MyNode.prototype.onDeselected = function()
{
	var children = this.getOutputNodes(0);
	if(!children) return;
	children.sort(function(a,b)
	{
		if(a.pos[0] > b.pos[0])
		{
		  return 1;
		}
		if(a.pos[0] < b.pos[0])
		{
		  return -1;
		}
	});

	this.outputs[0].links = [];
	for(var i in children)
		this.outputs[0].links.push(children[i].inputs[0].link);
}

//method called on the tree iteration
MyNode.prototype.tick = function(agent, dt)
{
    //get child nodes
	var children = this.getOutputNodes(0);
  
	for(var n in children)
	{
		var child = children[n];
      //tick every child to know if it succeeds, keeps running or fails 
      //value contains the status of the execution, the type of behaviour and the data to 
      //be applied according to the returned behavviour
		var value = child.tick(agent, dt);
		if(value && value.STATUS == STATUS.success)
		{
			//do whatever when child succeds
			return value;
		}
		else if(value && value.STATUS == STATUS.running)
		{
			//do whatever when child is running
			return value;
		}
	}

	if(this.running_node_in_banch)
			agent.bt_info.running_node_index = null;
      
  //in case child fails
	this.graph.current_behaviour.STATUS = STATUS.fail;
	return this.graph.current_behaviour;
}


MyNode.prototype.onConfigure = function(info)
{
  //method called when the node is loaded
}

//method called in case it has left input or right output to compute properties 
//to be used in the tick method (for evaluation i.e.)
MyNode.prototype.onExecute = function(info)
{
  //do whatever in execution
}

MyNode.title = "MyNode";
MyNode.desc = "MyNode description";

//register
LiteGraph.registerNodeType("btree/MyNode", MyNode);
```

The tick method is called for all the nodes of the Hybrid Behaviour Trees except for those which do not have a top connections. In other words, if the node just outputs some informations to another node, the tick method is not called. 

The onExecute method is called when there is an execution required, like some calculations for inner properties of the node, or to output the computation to other nodes. 


# Architecture of the library

HBTree.js contains two main classes; HBTContext and HBTGraph. 

HBTContext is the one in charge of knowing in which context an agent is being evaluated through a specific graph. Hence, it is in charge of handling the Interest Points of the scene, the Blackboard of the context, containing relevant information or possible paths. Moreover, the HBTContext class contains the method which must be called to evaluate what a certain agent must perform according to a particular graph in this context.

HBTGraph contains all the structure of the tree and the information of each node. This class contains the method called by the evaluate method form HBTContext; the runBehaviour. This is the method which has to be called to evaluate the behaviour of an agent, and the one which will output that behaviour to use in external modules.

