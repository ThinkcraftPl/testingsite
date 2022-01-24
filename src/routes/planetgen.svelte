<script>
	import * as THREE from 'three'
	import seedrandom from 'seedrandom'
	import {onMount} from 'svelte'
import { text } from 'svelte/internal';
	let scene=null,camera=null,renderer=null,geometry=null,material=null,sphere=null;
	let continents=new Array(), mapW=200, mapH=100;
	let continentCenters = new Array();
	let continentNum=10,stringSeed;
	let generator = seedrandom(stringSeed);
	let texture = null;
	function getRandomInt(min,max){
		return Math.round((generator() * (max - min) )) + min;
	}
	function newSeed(){
		stringSeed=''+Math.round(Math.random()*1000000);
		seedUpdated();
	}
	function seedUpdated(){
		generator = seedrandom(stringSeed);
	}
	function continentTexture(){
		let canvas = document.getElementById('texture');
		//let canvas = document.createElement('canvas');
		let ctx= canvas.getContext('2d');
		canvas.width=mapW;
		canvas.height=mapH;
		let imageData = ctx.createImageData(mapW,mapH);
		let data = imageData.data;
		for(let i =0; i<mapW; i++)
			for(let j =0; j<mapH; j++){
				let red = 4*(i+j*mapW);
				let x = (continents[i][j]) / (continentNum-1) * 255;
				data[red]=x;
				data[red+1]=0;
				data[red+2]=255-x;
				data[red+3]=255;
			}
		for(let i =0; i<continentNum; i++){
			let x=continentCenters[i][0];
			let y=continentCenters[i][1];
			let red = 4*(x+y*mapW);
			data[red]=0;
			data[red+1]=255;
			data[red+2]=0;
			data[red+3]=255;
		}
		//console.log(imageData,continents)
		ctx.putImageData(imageData,0,0);
		texture=new THREE.Texture(imageData);
		material = new THREE.MeshBasicMaterial({
			//color:0x00ff00
			map: texture
		});
		sphere.material=material;
		sphere.material.map.needsUpdate=true;
		sphere.material.needsUpdate=true;
	}
	function regenerate(){
		seedUpdated();
		for(let i =0; i<mapW; i++){
			continents[i]=new Array();
			for(let j=0; j<mapH; j++)
				continents[i][j]= null;
		}
		let queue = new Array();
		for(let i =0; i<continentNum; i++)
		{
			continentCenters[i]=new Array(getRandomInt(0,mapW),getRandomInt(0,mapH));
			queue.push(new Array(continentCenters[i],i));
		}
		while(queue.length!=0){
			let random = getRandomInt(0,queue.length-1);
			let now=queue[random];
			queue.splice(random,1);
			now[0][0]=(now[0][0]<0?mapW-1:(now[0][0]>=mapW?0:now[0][0]))
			now[0][1]=(now[0][1]<0?mapH-1:(now[0][1]>=mapH?0:now[0][1]))
			if(continents[now[0][0]][now[0][1]]==null)
			{
				continents[now[0][0]][now[0][1]]=now[1];
				queue.push(new Array(new Array(now[0][0],now[0][1]+1),now[1]));
				queue.push(new Array(new Array(now[0][0],now[0][1]-1),now[1]));
				queue.push(new Array(new Array(now[0][0]-1,now[0][1]),now[1]));
				queue.push(new Array(new Array(now[0][0]+1,now[0][1]),now[1]));
			}
		}
		continentTexture();
	}
	onMount(()=>{
		texture=new THREE.TextureLoader().load('./src/images/earth.jpg');
		let canvas=document.getElementById('viewport');
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera( 75, viewport.clientWidth/viewport.clientHeight, 0.1, 1000 );
		renderer = new THREE.WebGLRenderer({
			canvas,
			antialias:true,
		});
		geometry = new THREE.SphereGeometry(2,32,32);
		material = new THREE.MeshBasicMaterial({
			//color:0x00ff00
			map: texture
		});
		sphere = new THREE.Mesh( geometry, material );
		scene.add( sphere );
		camera.position.z = 5;
		newSeed();
		animate();
	});
	function animate() {
		requestAnimationFrame( animate );
		sphere.rotation.y += 0.01;
		
		renderer.setSize((window.innerWidth-96)*(sidebarHidden?0.99:0.84),window.innerHeight,false);
		const canvas = renderer.domElement;
		camera.aspect = canvas.clientWidth / canvas.clientHeight;
		camera.updateProjectionMatrix();
		renderer.render( scene, camera );
	}

	let sidebarHidden=false;
	function toggleSidebar(){
		sidebarHidden=!sidebarHidden;
		return false;
	}
</script>
<div class="main_wrapper {sidebarHidden===true?'sidebar-hidden':''}">
	<div class="sidebar" style="visibility: {sidebarHidden?"hidden":"visible"}">
		<div>
			<h2>Options:</h2>
		</div>
		<div>
			<label for="continents">Number of continents (1-20)</label>
			<br>
			<input bind:value={continentNum} type="range" min="1" max="20" class="slider" id="continents" on:change="{regenerate}">
			<br>
			{continentNum}
		</div>
		<div>
			<label for="seed">Seed (alphanumeric):</label>
			<br>
			<input bind:value={stringSeed} type="input" class="input" id="seed" on:change="{seedUpdated}">
			<br>
			<button class="invisible_button" on:click="{newSeed}">Get Random</button>
		</div>
		<div>
			<button class="button" on:click="{regenerate}">Generate</button>
		</div>
		<div>
			<canvas style="width: 200px; height:100px" id="texture"></canvas>
		</div>
	</div>
	<button class="sidebar-button" on:click={toggleSidebar}>
	</button>
	<canvas id="viewport" style="width: 100%; height: 100%; display:block; margin: 0;">
	</canvas>
</div>
<style>
	:global(body){
		
		margin: 0;
		height: 100%;
	}
	.main_wrapper{
		display: grid;
		grid-column-gap:0;
		grid-template-rows: minmax(100vh,max-content);
		grid-template-columns: 15% 1% auto;

		background-color: #333333;
	}
	.main_wrapper.sidebar-hidden{
		grid-template-columns: 0% 1% auto;
	}
	.sidebar{
		color:white;
		background-color: #333333;
		display: grid;
		grid-gap: 0;
		grid-template-columns: 100%;
		grid-auto-rows: max-content;
		padding: 5px;
	}
	.sidebar>div{
		text-align: center;
		align-items: center;
		justify-content: center;
		width: 100%;
		border: 2px solid gray;
		padding: 0;
		margin: 0;
	}
	.slider{
		width: 75%;
	}
	.input{
		width: 75%;
		text-align: center;
		background-color: inherit;
		border: 2px solid black;
		color:white;
	}
	.button{
		margin: 10px;
		width: 75%;
		text-align: center;
		background-color: black;
		border: 2px solid black;
		color:white;
	}
	.button:hover{
		background-color: white;
		color:black;
	}
	.invisible_button{
		background-color: transparent;
		border: 0px solid transparent;
		margin: 0px;
		padding: 0px;
		color: white;
	}
	.invisible_button:hover{
		color:gray;
	}
	.sidebar-button{
		display: flex;
		border: 0px solid transparent;
		background-color: #111111;
		color:white;
		justify-content: center;
		align-items: center;
		text-align: center;
		padding: 0;
	}
</style>