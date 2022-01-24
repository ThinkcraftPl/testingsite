<script>
	import { getContext, onMount } from 'svelte';
	let c, ctx;
	let G=null,N=null; 	//graph array of arrays and number of nodes
	let P=null,R=null, BR=null; 	//array of positions, radious of nodes and radious of big circle
	let directional=null; //is current graph directional
	let CS= new Array;		//collision check stack
	let CH=null;		//is collision handler on
	let selNode=-1; 	//selected node
	let allow_loops_opt=false, directional_opt=false, edge_weight_opt=false, tree_opt=false, connectivity_opt=false, use_probability_opt=false, FO=null;
	function getRandomFloat(min,max){
		return (Math.random() * (max - min) ) + min;
	}
	function getRandomInt(min,max){
		return Math.round((Math.random() * (max - min) )) + min;
	}

	function setCanvas(canvas){
		//setup canvas on startup
		c=canvas;
		c.width = c.clientWidth;
		c.height = c.clientHeight;
		ctx=c.getContext('2d');
		let min=Math.min(c.width,c.height);
		BR=min*0.4;
		//add mouse listeners
		c.addEventListener("mousedown", function(e){handleMouseDown(e);});
		c.addEventListener("mouseup", function(e){handleMouseUpOrOut(e);});
		c.addEventListener("mouseout", function(e){handleMouseUpOrOut(e);});
		c.addEventListener("mousemove", function(e){
			//let t0= performance.now();
			handleMouseMove(e);
			//let t1= performance.now();
			//console.log('Time taken to execute move function:'+ (t1-t0) +' milliseconds');
		});
	}
	function refreshCanvas(){
		//refresh canvas on resize
		c.width = c.clientWidth;
		c.height = c.clientHeight;
		ctx=c.getContext('2d');
		let min=Math.min(c.width,c.height);
		BR=min*0.4;
	}
	function copyGraph(){
		//copy graph to clipboard
		//check if graph exists
		if(G==null){
			alert("no graph generated!");
			return false;
		}
		let output="";
		//iterate through graph array
		G.forEach((node,i)=>{
			//for each node find all connections
			node.forEach(connection=>{
				//add to output string
				output+=i+" "+connection[0]+" "+connection[1]+"\n";
			});
		});
		//copy output string to clipboard
		navigator.clipboard.writeText(output);
		return false;
	}
	function generateGraph(){
		if(use_probability_opt){
			let nodes = parseInt(document.getElementById('nodes').value);
			let probability = parseFloat(document.getElementById('probability').value);
			//validate input
			if(isNaN(nodes)||isNaN(probability))
			{
				drawMessage("Type number of nodes and probability!");
				return false;
			}
			console.log(nodes+" "+probability);

			N=nodes;
			G=new Array;
			for(let i =1; i<=N; i++)
				G[i]=new Array();
			for(let i =1; i<=N; i++){
				for(let j = (allow_loops_opt?i:i+1); j<=N; j++){
					if(getRandomFloat(0,1)<probability)
					{
						//add connection
						let temp = new Array()
						if(directional_opt)
						{
							if(getRandomFloat(0,1)<0.5)
							{
								temp.push(j);
								G[i].push(temp);
							}
							else
							{
								temp.push(i);
								G[j].push(temp);
							}
						}
						else
						{
							temp.push(j);
							G[i].push(temp); //connection info - node
						}
					}
				}
			}

			P=new Array();

			for(let i =1; i<=N; i++)
			{
				P[i]=new Array();
				P[i][0]=((2*Math.PI*i)/N);
				P[i][1]=1;
			}
			//console.log(G);
			//console.log(P);
			//draw graph
			drawGraph();
		}else{
			let nodes = parseInt(document.getElementById('nodes').value);
			let edges = parseInt(document.getElementById('probability').value);
			//validate input
			if(isNaN(nodes)||isNaN(edges))
			{
				drawMessage("Type number of nodes and edge number!");
				return false;
			}
			console.log(nodes+" "+edges);
			
			N=nodes;
			G=new Array;
			for(let i =1; i<=N; i++)
				G[i]=new Array();
			let possibleConnections = new Array();
			for(let i =1; i<=N; i++)
				for(let j=(allow_loops_opt?i:i+1); j<=N; j++)
					possibleConnections.push(new Array(i,j));
			for(let i=possibleConnections.length-1; i>=1; i--){
				let random = getRandomInt(0,i);
				let temp = possibleConnections[random];
				possibleConnections[random]=possibleConnections[i];
				possibleConnections[i]=temp;
			}
			//console.log(possibleConnections);
			for(let k =0; k<edges; k++){
				let i=possibleConnections[k][0];
				let j=possibleConnections[k][1];
				//add connection
				let temp = new Array()
				if(directional_opt)
				{
					if(getRandomFloat(0,1)<0.5)
					{
						temp.push(j);
						G[i].push(temp);
					}
					else
					{
						temp.push(i);
						G[j].push(temp);
					}
				}
				else
				{
					temp.push(j);
					G[i].push(temp); //connection info - node
				}
			}
			
			P=new Array();

			for(let i =1; i<=N; i++)
			{
				P[i]=new Array();
				P[i][0]=((2*Math.PI*i)/N);
				P[i][1]=1;
			}
			//console.log(G);
			//console.log(P);
			//draw graph
			drawGraph();
		}
		// //get number of nodes, probability, and weight ranges from input boxes
		// let nodes = parseInt(document.getElementById('nodes').value);
		// let probability = parseFloat(document.getElementById('probability').value);
		// let minWeight = parseInt(document.getElementById('min-weight').value)
		// let maxWeight = parseInt(document.getElementById('max-weight').value)
		// //validate input
		// if(isNaN(nodes)||isNaN(probability))
		// {
		// 	drawMessage("Type number of nodes and probability!");
		// 	return false;
		// }
		// if(isNaN(minWeight)||isNaN(maxWeight))
		// {
		// 	minWeight=0;
		// 	maxWeight=0;
		// }
		// console.log(nodes+" "+probability);
		// //create graph structure
		// let graph = new Array();
		// for(let i =1; i<=nodes; i++)
		// 	graph[i]=new Array();
		// //for each node get possible connections
		// for(let i =0; i<nodes; i++)
		// {
		// 	for(let j =i; j<nodes; j++)
		// 	{
		// 		if(i!=j){
		// 			//calculate probability and check if connection is made
		// 			if(getRandomFloat(0,1)<probability)
		// 			{
		// 				//add connection
		// 				graph[i+1].push(new Array(j+1,getRandomInt(minWeight,maxWeight))); //connection info - node, weight
		// 			}
		// 		}
		// 	}
		// }
		// //save to global variables
		// G=graph;
		// N=nodes;
		// //for each node get position and save
		// P=new Array();
		// for(let i =1; i<=nodes; i++)
		// {
		// 	P[i]=new Array();
		// 	P[i][0]=((2*Math.PI*i)/N);
		// 	P[i][1]=1;
		// }
		// //draw graph
		// drawGraph();
		// return false;
	}
	//get cords from angle and % of radius
	function getCords(node, radious){
		if(node==-1)
			return new Array((c.width/2),(c.height/2));
		return new Array((c.width/2)+(radious*P[node][1]*Math.cos(P[node][0])),(c.height/2)+(radious*P[node][1]*Math.sin(P[node][0])));
	}
	//find angle and % of radius from cords
	function findCords(pos, radious){
		let angle = Math.atan((pos[1]-c.height/2)/(pos[0]-c.width/2));
		if(angle<0)
			angle+=2*Math.PI;
		let percent = (pos[0]-c.width/2)/(radious*Math.cos(angle));
		if(percent<0)
		{
			percent=0-percent;
			angle+=Math.PI;
			if(angle>2*Math.PI)
				angle-=2*Math.PI;
		}
		return new Array(angle,percent);
	}
	function drawGraph(){
		//clear canvas
		ctx.clearRect(0, 0, c.width, c.height);
		//check if graph exists
		if(G==null){
			return false;
		}
		//console.log("Draw");
		//get minimum of canvas height and width
		const min = Math.min(c.height, c.width);
		//draw edges
		//iterate through graph array
		//get radious of nodes
		let radi=(Math.PI*BR*(N==1?0.1:((1-1/Math.sqrt(N))/2)))/(N);
		R=radi;
		G.forEach((node,i)=>{
			//for each node find all connections
			node.forEach(connection=>{
				//get connected node's index
				let j = connection[0];
				//console.log(i,j)
				if(i==j){
					//get positions
					let posi=getCords(i,BR);
					let center=getCords(-1,0);
					//console.log(posi,posj);
					//calculate point for quadratic curve
					let vector = new Array(center[0]-posi[0],center[1]-posi[1])
					vector = new Array(vector[0]/getDistance(posi,center),vector[1]/getDistance(posi,center))
					let cpoint = new Array(posi[0]-vector[0]*R,posi[1]-vector[1]*R);
					ctx.beginPath();
					ctx.fillStyle='black';
					ctx.strokeStyle='black';
					ctx.arc(cpoint[0],cpoint[1],R,0,2*Math.PI,false);
					ctx.stroke();
				}
				else{
					//get positions
					let posi=getCords(i,BR);
					let posj=getCords(j,BR);
					//console.log(posi,posj);
					//calculate point for quadratic curve
					let cpoint = new Array();
					cpoint.push(((c.width/2)+((posi[0]+posj[0])/2))/2);
					cpoint.push(((c.height/2)+((posi[1]+posj[1])/2))/2);
					//draw quadratic curve from i to j
					ctx.beginPath();
					ctx.fillStyle='black';
					ctx.strokeStyle='black';
					ctx.moveTo(posi[0],posi[1]);
					ctx.quadraticCurveTo(cpoint[0],cpoint[1],posj[0],posj[1]);
					ctx.stroke();
				}
			});
		});
		//draw nodes
		//start drawing and set style
		ctx.fillStyle='black';
		ctx.strokeStyle='black';
		//iterate through nodes
		for(let i =1; i<=N; i++){
			//get cords of node
			let cords = getCords(i,BR)
			
			//draw circle
			ctx.beginPath();
			ctx.arc(cords[0],cords[1],radi,0,2*Math.PI,false);
			ctx.fill();
			ctx.stroke();
		}
		//set style for text
		ctx.fillStyle='white';
		ctx.strokeStyle='white';
		ctx.textAlign='center';
		ctx.textBaseline='middle';
		ctx.font='30px Arial';
		ctx.font=Math.floor(((radi)/ctx.measureText(N).width)*30)+'px Arial';
		//iterate through nodes
		for(let i =1; i<=N; i++){
			//get cords of node
			let cords = getCords(i,BR)
			
			//draw text
			ctx.fillText(i,cords[0],cords[1])
		}
	}

	//Draw a string on canvas
	function drawMessage(message){
		ctx.clearRect(0, 0, c.width, c.height);			//clear canvas
		ctx.font="30px Arial";							//set font
		ctx.textAlign="center";							//set align
		ctx.fillText(message,c.width/2,c.height/2);		//draw text
	}

	//function get distance between two points
	function getDistance(posi,posj){
		return Math.sqrt((Math.pow(posi[0]-posj[0],2)+Math.pow(posi[1]-posj[1],2)));
	}

	//Handle Mouse things

	//get mouse pos
	/*
	function getMousePosition(event) {
		let rect = c.getBoundingClientRect();
		let x = event.clientX - rect.left;
		let y = event.clientY - rect.top;
		return new Array(x,y);
	}
	*/
	function getMousePosition(evt) {
		var rect = c.getBoundingClientRect();
		return new Array(
			(evt.clientX - rect.left) / (rect.right - rect.left) * c.width,
			(evt.clientY - rect.top) / (rect.bottom - rect.top) * c.height
		);
	}

	//on click
	function handleMouseDown(e){
		e.preventDefault();
		
		//get minimum of canvas height and width
		const min = Math.min(c.height, c.width);

		let pos = getMousePosition(e);
		//if graph or position array is null exit
		if(G==null||P==null)
			return false;
		//iterate through nodes and see if mouse clicked on one
		for(let node = 1; node<=N; node++){
			//console.log(getCords(node,BR));
			if(getDistance(getCords(node,BR),pos)<=R)
			{
				selNode=node;
				break;
			}
		}
		console.log(selNode);
		console.log("down");
	}
	function collisionHandling(){
		let current=CS[0];
		//check if current is invalid
		//console.log(CS[0]);
		if(current==-1)
			CS.shift();
		else
		{
			//if position array is null, clear queue
			if(P==null)
				CS=new Array();
			let collisions=false;
			let posc=getCords(current,BR);
			for(let i =1; i<=N; i++){
				if(i!=current&&i!=selNode)
				{

					let posi=getCords(i,BR);
					let dist = getDistance(posi,posc);
					if(dist<=2*R)
					{
						collisions=true;
						let newPos = new Array(posi[0]+(R/10)*((posi[0]-posc[0])/dist),posi[1]+(R/10)*((posi[1]-posc[1])/dist));
						//console.log((posi[0]/dist));
						P[i]=findCords(newPos,BR);
						drawGraph();
						CS.push(i);
					}
				}
			}
			if(!collisions)
				CS.shift();
		}
		//if no nodes to check in queue, stop handler
		if(CS.length==0)
		{
			clearInterval(CH);
			CH=null;
		}
	}
	//done dragging
	function handleMouseUpOrOut(e){
		e.preventDefault();
		//handle collisions
		CS.push(selNode);
		if(CH==null)
			CH=setInterval(collisionHandling,10);
		//deselect nodes
		selNode=-1;
		console.log("up or out");
	}
	//on move
	function handleMouseMove(e){
		e.preventDefault();
		//if no node selected, return
		if(selNode==-1)
		{
			/*
			let pos = getMousePosition(e);
			ctx.beginPath();
			ctx.arc(pos[0],pos[1],1,0,2*Math.PI,false);
			ctx.fill();
			ctx.stroke();
			*/
			return false;
		}
		
		//get minimum of canvas height and width
		const min = Math.min(c.height, c.width);

		//if a node is selected get mouse position and set node position to mouse pos
		let pos = getMousePosition(e);

		P[selNode]=findCords(pos,BR);
		//console.log(P[selNode]);
		//refresh graph
		drawGraph();
		console.log("move");
	}
	//space out
	function spaceOut(){
		//if no number of nodes specified or Position array null, return
		if(N==null||P==null)
			return false;
		let PSorted = new Array();
		P.forEach((node,i)=>{
			PSorted.push(new Array(node[0],node[1],i));
		});
		PSorted.sort(function(a, b){return a[0]>=b[0]});
		PSorted.unshift();
		//console.log(PSorted);
		//console.log(P);
		PSorted.forEach((node,i)=>{
			//console.log(node[2],i);
			P[node[2]]=new Array();
			P[node[2]][0]=((2*Math.PI*(i+1))/N);
			P[node[2]][1]=1;
		});
		drawGraph();
	}
	//reset view
	function resetView(){
		//if no number of nodes specified, return
		if(N==null)
			return false;
		//clear position array and reset
		P=new Array();
		for(let i =1; i<=N; i++)
		{
			P[i]=new Array();
			P[i][0]=((2*Math.PI*i)/N);
			P[i][1]=1;
		}
		drawGraph();
	}
	function fetchOptions(){
		let prob_min=0, prob_max=1, prob_whole;
		allow_loops_opt		= document.getElementById('allow_loops').checked;
		directional_opt		= document.getElementById('directional').checked;
		edge_weight_opt		= document.getElementById('edge_weight').checked;
		tree_opt			= document.getElementById('tree').checked;
		connectivity_opt	= document.getElementById('connectivity').checked;
		use_probability_opt = document.getElementById('use_probability').checked;
		if(use_probability_opt){
			document.getElementById("probability_text").innerHTML="Probability:<br>";
			prob_min=0;
			prob_max=1;
			prob_whole=false;
		}else{
			document.getElementById("probability_text").innerHTML="Number of edges:<br>";
			let n = document.getElementById("nodes").value;
			prob_min=1;
			prob_max=(n*(n-1))/2;
			prob_whole=true;
		}
		if(connectivity_opt)
		{
			let n = document.getElementById("nodes").value;
			prob_min=n-1;
			document.getElementById('use_probability').checked=false;
			document.getElementById('tree').checked=false;
		}
		if(tree_opt)
		{
			document.getElementById('use_probability').checked=false;
			document.getElementById('connectivity').checked=false;
			document.getElementById('allow_loops').checked=false;

		}
		if(use_probability_opt)
		{
			document.getElementById('connectivity').checked=false;
			document.getElementById('tree').checked=false;

		}
		if(document.getElementById("probability").value<prob_min)
			document.getElementById("probability").value=prob_min;
		else if(document.getElementById("probability").value>prob_max)
			document.getElementById("probability").value=prob_max;
		if(prob_whole)
		{
			document.getElementById("probability").value=parseInt(document.getElementById("probability").value);
		}
	}
	onMount(()=>{
		const c = document.getElementById("responsive-canvas");

		setCanvas(c);
		//add resize listnener
	
		window.addEventListener('resize', () => {
    		refreshCanvas();
			drawGraph();
    	});
		if(FO==null)
			FO=setInterval(fetchOptions,10);
	});
	
</script>

<canvas id="responsive-canvas"></canvas>
<div class="opcje">
	<div class = "grid-wrapper">
		<div class="input-div">
			<span id="node_text">Number of nodes:<br></span>
			<input type="number" id="nodes" class="number-selector" value="10">
		</div>
		<div class="input-div">
			<span id="probability_text">Probability:<br></span>
			<input type="number" id="probability" class="number-selector" disabled={tree_opt} value="0.2">
		</div>
		<div class="input-div">
			<span id="minweight_text">Min weight:<br></span>
			<input type="number" id="min-weight" class="number-selector" disabled={!edge_weight_opt}>
		</div>
		<div class="input-div">
			<span id="maxweight_text">Max weight:<br></span>
			<input type="number" id="max-weight" class="number-selector" disabled={!edge_weight_opt}>
		</div>
		<button id="submit" class="submit-input" on:click|preventDefault={generateGraph}>Generate</button>
		<div class="options">
			<div>
				<input type="checkbox" id="allow_loops" name="allow_loops" value="allow_loops" disabled={tree_opt}>
				<label for="allow_loops">Allow Loops</label>
			</div>
			<div>
				<input type="checkbox" id="directional" name="directional" value="directional" >
				<label for="directional">Directional</label>
			</div>
			<div>
				<input type="checkbox" id="edge_weight" name="edge_weight" value="edge_weight" >
				<label for="edge_weight">Edge weight</label>
			</div>
			<div>
				<input type="checkbox" id="tree" name="tree" value="tree" disabled={connectivity_opt||use_probability_opt}>
				<label for="tree">Generate Tree</label>
			</div>
			<div>
				<input type="checkbox" id="connectivity" name="connectivity" value="connectivity" disabled={tree_opt||use_probability_opt}>
				<label for="connectivity">Force connectivity</label>
			</div>
			<div>
				<input type="checkbox" id="use_probability" name="use_probability" value="use_probability" disabled={connectivity_opt||tree_opt}>
				<label for="use_probability">Use probability</label>
			</div>
			<div style="grid-column:span 2">
				<span>
					Note: Some of above options are mutually exclusive!
				</span>
			</div>
		</div>
		<button id="submit" class="interaction-button" on:click|preventDefault={copyGraph}>Copy Graph</button>
		<button id="submit" class="interaction-button" on:click|preventDefault={spaceOut}>Space Out</button>
		<button id="submit" class="interaction-button" on:click|preventDefault={resetView}>Reset View</button>
	</div>
</div>
<style>
	/* Chrome, Safari, Edge, Opera */
	input::-webkit-outer-spin-button,
	input::-webkit-inner-spin-button {
	-webkit-appearance: none;
	margin: 0;
	}
	/* Firefox */
	input[type=number] {
		-moz-appearance: textfield;
	}
	.submit-input{
		text-align: center;
		background-color: black;
		border: 0px solid transparent;
		color: white;
		border-radius: 10px;
		padding: 5px;
		width: 100%;
		height: 100%;
		transition-property: background-color, border-color, color, fill, stroke, border-radius;
    	transition-timing-function: linear;
    	transition-duration: 150ms;
	}
	.submit-input:hover{
		background-color: white;
		color: black;
	}
	.interaction-button{
		text-align: center;
		background-color: black;
		border: 0px solid transparent;
		color: white;
		border-radius: 10px;
		padding: 5px;
		width: 100%;
		height: 100%;
		transition-property: background-color, border-color, color, fill, stroke, border-radius;
    	transition-timing-function: linear;
    	transition-duration: 150ms;
	}
	.interaction-button:hover{
		background-color: white;
		color: black;
	}
	.input-div{
		text-align: center;
		background-color: black;
		border: 0px solid transparent;
		color: white;
		border-radius: 10px;
		padding: 5px;
		width: 100%;
		height: max-content;
	}
	.options{
		display: grid;
		grid-column-gap:4%;
		grid-row-gap:20px;
		grid-auto-rows: auto max-content;
		grid-template-columns: repeat(4,22%);
		text-align: center;
		background-color: black;
		border: 0px solid transparent;
		color: white;
		border-radius: 10px;
		padding-left: 5px;
		padding-right: 5px;
		padding-top: 0px;
		padding-bottom: 0px;
		width: 100%;
		height: 100%;
		grid-column: span 4;
		grid-row: span 3;
	}
	.options > div {
		height: fit-content;
		padding: 10px;
	}
	.number-selector{
		background-color: black;
		border: 3px solid gray;
		border-radius: 3px;
		color:white;
		text-decoration: none;
		width:80%;
	}
	.number-selector:disabled{
		background-color: white;
	}
	.opcje{
		margin: 0;
	}
	.grid-wrapper{
		display:grid;
		grid-template-columns: repeat(5,18%);
		grid-template-rows: max-content;
		grid-row-gap:10px;
		grid-column-gap:2%;
		padding-left:1%;
	}
	canvas#responsive-canvas{
		margin: 0;
		width: 100%;
		height: 70%;
	}
</style>