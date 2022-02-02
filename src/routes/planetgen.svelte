<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import seedrandom from 'seedrandom'
import {onMount, onDestroy} from 'svelte'
let scene=null,camera=null,renderer=null,geometry=null,material=null,sphere=null,controls=null;
let tecPlaCube=new Array(), cmapD=512; // tectonic plates cube projection (array of up - 0,down - 1,left - 2,front - 3,right - 4,back - 5)
let mapW=cmapD*2, mapH=cmapD; // tectonic plates equi projection
let tectonicPlateCenters = new Array(); // face, position
let tectonicPlateInfo = new Array() // angle, velocity, type (false - oceanic, true - continental)
let tectonicPlateCollisions = new Array();
let tectonicPlateNum=10,stringSeed,oceanicPlateNum=4;
let generationReport="Not generating!"
let generator = seedrandom(stringSeed);
let chunkDoer=null;
let texture = null;
onDestroy(()=>{
	clearTimeout(chunkDoer);
})
function getRandomInt(min,max){
	min = Math.ceil(min);
  	max = Math.floor(max);
	if(min==max)
		return min;
  	return Math.floor(generator() * (max - min + 1) + min);
}
function getRandomFloat(min,max){
	return (generator() * (max - min) ) + min;
}
function newSeed(){
	stringSeed=''+Math.round(generator()*1000000);
	seedUpdated();
}
function seedUpdated(){
}
function displayCubemap(cubeArray, faceSize, mapWidth, mapHeight){
	let canvas = document.getElementById('equiv-out');
	//let canvas = document.createElement('canvas');
	let ctx= canvas.getContext('2d');
	canvas.width=mapW;
	canvas.height=mapH;
	let imageData = ctx.createImageData(mapW,mapH);
	let outputArray = imageData.data;
	let u,v,phi,theta;
//let faceDebug = new Array(0,0,0,0,0,0);
	let i=0,j=0;
	function doChunk(){
		let start = new Date().getTime();
		while(j<mapHeight&&((new Date().getTime())-start)<=50)
		{
			v=1-(j/mapHeight);
			theta=v*Math.PI;
			if(i>=mapWidth)
				i=0;
			while(i<mapWidth&&((new Date().getTime())-start)<=50)
			{
				u=i/mapWidth;
				phi=2*u*Math.PI;
				let x,y,z;
				x = Math.sin(phi) * Math.sin(theta) * -1;
				y = Math.cos(theta);
				z = Math.cos(phi) * Math.sin(theta) * -1;
				let xa,ya,za;
				let a = Math.max(Math.abs(x),Math.abs(y),Math.abs(z));
				xa=x/a;
				ya=y/a;
				za=z/a;
				let xPixel,yPixel,face;
				if(xa==1)
				{
					face=4;
					xPixel = Math.round((((za+1)/2)-1)*(faceSize-1));
					yPixel = Math.round((((ya+1)/2))*(faceSize-1));
				}
				else if(xa==-1)
				{
					face=2;
					xPixel = Math.round((((za+1)/2))*(faceSize-1));
					yPixel = Math.round((((ya+1)/2))*(faceSize-1));
				}
				else if(ya==1)
				{
					face=0;
					xPixel = Math.round((((xa+1)/2))*(faceSize-1));
					yPixel = Math.round((((za+1)/2)-1)*(faceSize-1));
				}
				else if(ya==-1)
				{
					face=1;
					xPixel = Math.round((((xa+1)/2))*(faceSize-1));
					yPixel = Math.round((((za+1)/2))*(faceSize-1));
				}
				else if(za==1)
				{
					face=3;
					xPixel = Math.round((((xa+1)/2))*(faceSize-1));
					yPixel = Math.round((((ya+1)/2))*(faceSize-1));
				}
				else if(za==-1)
				{
					face=5;
					xPixel = Math.round((((xa+1)/2)-1)*(faceSize-1));
					yPixel = Math.round((((ya+1)/2))*(faceSize-1));
				}
				xPixel=Math.abs(xPixel);
				yPixel=Math.abs(yPixel);
				
				let equivIndex = 4*(i+(mapHeight-j)*mapWidth);
				let cubeIndex = 4*(xPixel+(faceSize-1-yPixel)*faceSize);
				if(xPixel>=faceSize||faceSize-1-yPixel>=faceSize)
				{
					//console.log(xa,ya,za,xPixel,yPixel)
					outputArray[equivIndex]=255;
					outputArray[equivIndex+1]=0;
					outputArray[equivIndex+2]=0;
					outputArray[equivIndex+3]=255;
					//faceDebug[face]++;
				}
				// else if(cubeArray[face][cubeIndex]==undefined)
				// {
					// 	outputArray[equivIndex]=0;
					// 	outputArray[equivIndex+1]=255;
					// 	outputArray[equivIndex+2]=0;
					// 	outputArray[equivIndex+3]=255;
					// }
				else
				{
					for(let k = 0; k<4; k++)
					outputArray[equivIndex+k]=cubeArray[face][cubeIndex+k];
				}
				i++;
			}
			if(i>=mapWidth)
				j++;
		}
		if(i<mapWidth||j<mapHeight){
			generationReport="Converting cubemap to equirectangular: "+Math.round(100*((j)/(mapHeight)))+"%";
			chunkDoer=setTimeout(doChunk,1);
		}
		else
		{
			generationReport="Not generating!";
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
	}
	doChunk();
//console.log(faceDebug);
}
function tectonicPlateTexture(){
	
	let cubeArray=new Array();
	for(let f=0; f<6;f++)
		cubeArray[f]=new Array();
	let f=0,index=0;
	let iterationsDone=0;
	function doChunk(){
		let start = new Date().getTime();
		while(f<6&&((new Date().getTime())-start)<=50)
		{
			if(index>4*(cmapD-1)*(cmapD+1))
			{
				index=0;
			}
			while(index<=4*(cmapD-1)*(cmapD+1)&&((new Date().getTime())-start)<=50)
			{
				let i=Math.round((index/4))%cmapD;
				let j=Math.round(((index/4)-i)/cmapD);
				let x = Math.round((tecPlaCube[f][i][j]) / (tectonicPlateNum-1) * 150)+105;
				cubeArray[f][index]=0;
				cubeArray[f][index+1]=tectonicPlateInfo[tecPlaCube[f][i][j]][2]?x:0;
				cubeArray[f][index+2]=!tectonicPlateInfo[tecPlaCube[f][i][j]][2]?x:0;
				cubeArray[f][index+3]=255;
				iterationsDone++;
				index+=4;
			}
			if(index>4*(cmapD-1)*(cmapD+1))
			{
				f++;
			}
		}
		if(f<6)
		{
			generationReport="Generating tectonic cubemap: "+Math.round(100*(f/6)+100*((index)/(6*4*(cmapD-1)*(cmapD+1))))+"%";
			chunkDoer=setTimeout(doChunk,1);
			return;
		}
		else
		{
			setTimeout(displayCubemap(cubeArray,cmapD,mapW,mapH),1);
		}
	}
	doChunk();
	// for(let f =0; f<6; f++){
	// 	let ca = document.getElementById('cube-'+f);
	// 	let cont=ca.getContext('2d');
	// 	ca.width=cmapD;
	// 	ca.height=cmapD;
	// 	let imgData = cont.createImageData(cmapD,cmapD);
	// 	let dat = imgData.data;
	// 	for(let i =0; i<cmapD; i++)
	// 		for(let j =0; j<cmapD; j++){
	// 			let index = 4*(i+j*cmapD);
	// 			for(let k =0; k<4; k++)
	// 				dat[index+k]=cubeArray[f][index+k];
	// 		}
	// 	cont.putImageData(imgData,0,0);
	// }
	//console.log(imageData)
}
function regenerate(){
	generator = seedrandom(stringSeed);
	for(let i =0; i<6; i++){
		tecPlaCube[i]=new Array();
		for(let j =0; j<cmapD; j++)
			tecPlaCube[i][j]=new Array(cmapD);
	}
	clearTimeout(chunkDoer);
	chunkDoer=null;
	generateTectonics();
}
function rotateCords(pos,angle){ //rotate cords by multiples of 90 degrees
	let ang=Math.round(angle/90);
	ang=ang<0?ang+4:ang;

	let t = pos[0];
	pos[0]=(cmapD-1)-pos[1];
	pos[1]=t;

	if(ang==1){
		return pos;
	}
	else
	{
		return rotateCords(pos,(ang-1)*90);
	}
}
function validateCubeMapCords(face,pos){
	let modified=false;
	if(pos[0]>=cmapD){
		modified=true;
		pos[0]-=cmapD;
		switch(face){
			case 0:
				face=4;
				pos=rotateCords(pos,90);
				break;
			case 1:
				face=4;
				pos=rotateCords(pos,270);
				break;
			case 2:
			case 3:
			case 4:
				face++;
				break;
			case 5:
				face=2;
				break;
		}
	}
	else if(pos[0]<0){
		modified=true;
		pos[0]=cmapD+pos[0];
		switch(face){
			case 0:
				face=2;
				pos=rotateCords(pos,270);
				break;
			case 1:
				face=2;
				pos=rotateCords(pos,90);
				break;
			case 2:
				face=5;
				break;
			case 3:
			case 4:
			case 5:
				face--;
				break;
		}
	}
	if(pos[1]>=cmapD){
		modified=true;
		pos[1]-=cmapD;
		switch(face){
			case 0:
				face=3;
				break;
			case 1:
				face=5;
				pos=rotateCords(pos,180);
				break;
			case 2:
				face=1;
				pos=rotateCords(pos,270);
				break;
			case 3:
				face=1;
				break;
			case 4:
				face=1;
				pos=rotateCords(pos,90);
				break;
			case 5:
				face=1;
				pos=rotateCords(pos,180);
				break;
		}
	}
	else if(pos[1]<0){
		modified=true;
		pos[1]=cmapD+pos[1];
		switch(face){
			case 0:
				face=5;
				pos=rotateCords(pos,180);
				break;
			case 1:
				face=3;
				break;
			case 2:
				face=0;
				pos=rotateCords(pos,90);
				break;
			case 3:
				face=0;
				break;
			case 4:
				face=0;
				pos=rotateCords(pos,270);
				break;
			case 5:
				face=0;
				pos=rotateCords(pos,180);
				break;
		}
	}
	return new Array(face,pos);
}
function generateTectonics()
{
	const queue = new Array(); //tectonic plate number, face, position
	let lastpos=0;
	for(let i =0; i<tectonicPlateNum; i++)
	{
		let pos = new Array(getRandomInt(0,cmapD),getRandomInt(0,cmapD));
		tectonicPlateInfo[i] = new Array(getRandomFloat(0, Math.PI*2),getRandomFloat(0,1),true);
		let face = getRandomInt(0,5);
		tectonicPlateCenters[i]=new Array(face,pos);
		queue[lastpos]=new Array(i,face,pos);
		lastpos++;
	}
	const randomizePlates = new Array(tectonicPlateNum);
	for(let i =0; i<tectonicPlateNum; i++)
		randomizePlates[i]=i;
	for(let i =tectonicPlateNum-1; i>=0; i--)
	{
		let random = getRandomInt(0,i);
		let temp = randomizePlates[i];
		randomizePlates[i]=randomizePlates[random];
		randomizePlates[random]=temp;
	}
	for(let i=0; i<oceanicPlateNum; i++)
	{
		tectonicPlateInfo[randomizePlates[i]][2]=false;
	}
	//console.log(tectonicPlateInfo);
	let iterationsDone=0;
	let startP = performance.now();
	function doChunk(){
		let start = new Date().getTime();
		while(lastpos!=0&&((new Date().getTime())-start)<=50){
			iterationsDone++;
			let random = getRandomInt(0,lastpos-1);
			//let random =0;
			let now = queue[random];
			queue[random]=queue[lastpos-1];
			lastpos--;

			let face=now[1];
			let pos=now[2];
			let validate = validateCubeMapCords(face,pos);
			face=validate[0];
			pos=validate[1];
			if(tecPlaCube[face][pos[0]][pos[1]]==null){
				tecPlaCube[face][pos[0]][pos[1]]=now[0];
				queue[lastpos]=new Array(now[0],face,new Array(pos[0]+1,pos[1]));
				lastpos++;
				queue[lastpos]=new Array(now[0],face,new Array(pos[0]-1,pos[1]));
				lastpos++;
				queue[lastpos]=new Array(now[0],face,new Array(pos[0],pos[1]+1));
				lastpos++;
				queue[lastpos]=new Array(now[0],face,new Array(pos[0],pos[1]-1));
				lastpos++;
			}
		}
		if(lastpos!=0)
		{
			generationReport="Generating tectonic plates: "+Math.round(100*((iterationsDone)/(cmapD*cmapD*4*6)))+"%";
			chunkDoer = setTimeout(doChunk,10);
		}
		else
		{
			generateHeightmap();
			let endP = performance.now();
			console.log(endP-startP);
		}
	}
	doChunk();
}
function generateHeightmap()
{
	tectonicPlateTexture();
	generationReport="Not generating!";
}


onMount(()=>{
	texture=new THREE.TextureLoader().load('./images/earth.jpg');
	let canvas=document.getElementById('viewport');
	scene = new THREE.Scene();
	camera = new THREE.PerspectiveCamera( 75, viewport.clientWidth/viewport.clientHeight, 0.1, 1000 );

	
	renderer = new THREE.WebGLRenderer({
		canvas,
		antialias:true,
	});

	controls = new OrbitControls(camera, renderer.domElement);
	
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
			<label for="tectonicPlates">Number of Tectonic Plates (1-20)</label>
			<br>
			<input bind:value={tectonicPlateNum} type="range" min="1" max="20" class="slider" id="tectonicPlates">
			<br>
			{tectonicPlateNum}
		</div>
		<div>
			<label for="oceanicPlates">Number of Oceanic Plates (1-{tectonicPlateNum})</label>
			<br>
			<input bind:value={oceanicPlateNum} type="range" min="1" max={tectonicPlateNum} class="slider" id="oceanicPlates">
			<br>
			{oceanicPlateNum}
		</div>
		<div>
			<label for="seed">Seed (alphanumeric):</label>
			<br>
			<input bind:value={stringSeed} type="input" class="input" id="seed" on:change="{seedUpdated}">
			<br>
			<button class="invisible_button" on:click="{newSeed}">Get Random</button>
		</div>
		<div>
			<label for="cmapd">Cubemap face size (px):</label>
			<br>
			<input type="input" class="input" id="cmapd" bind:value="{cmapD}" on:change="{()=>{if(isNaN(parseInt(cmapD))){cmapD=512;mapW=512;mapH=256;}else{cmapD=parseInt(cmapD);mapW=cmapD*2;mapH=mapW}}}">
			<br>
		</div>
		<div>
			<button class="button" on:click="{regenerate}">Generate</button>
			<br>{generationReport}
		</div>
		
		<div>
			<canvas id="equiv-out" style="width:200px; height:100px"></canvas>
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
		margin:5px;
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