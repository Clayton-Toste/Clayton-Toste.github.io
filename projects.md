---
layout: page
title: Projects
permalink: /projects/
classes: wide
---

<style type="text/css">
.wrapper {
  max-width: 90%;
}
.column1 {
  width: 25%;
  margin-right: 75%;
  position: fixed;
}
  
.column2 {
  width: 75%;
  margin-left: 25%;
}
</style>

<div class="column1">
<h2>Table Contents</h2>
<ul>
  <li><a href="#sandlocked">Sandlocked</a></li>
  <li><a href="#deadweight">Dead Weight</a></li>
  <li><a href="#anderson">Home for Anderson</a></li>
  <li><a href="#atherborne">Atherborne</a></li>
  <li><a href="#sanchari">Sanchari</a></li>
  <li><a href="#pipe">Pipe Game</a></li>
  <li><a href="#jello">CS184 Final: Jello</a></li>
  <li><a href="#deblurred">Deblurred Diffusion</a></li>
  <li><a href="#shape">Shape-Guided Diffusion</a></li>
  <li><a href="#sa">Surface Analysis</a></li>
  <li><a href="#mole">Mole Game</a></li>
</ul>  
</div>

<div class="column2">
<p align=middle>
  <h2 id="sandlocked" align=middle><i>Sandlocked</i></h2>
</p>
<p align=middle>
  <img src="../gameplay2.png" width=500>
</p>
<p align=middle>
Sandlocked is a puzzle game developed in Unity. Solve puzzles by controlling a sand-moving drone and wrench-wielding mechanic. The game features a custom sand physics engine using cellular automaton and marching squares. Additionally, it implements a custom version of A* pathfinding that allows for finding the shortest path without adhering to a grid.
</p>
<p align=middle>
<iframe frameborder="0" src="https://itch.io/embed/2843541" width="552" height="167"><a href="https://pragma-twice.itch.io/sandlocked">Sandlocked by Pragma-Twice, nikkudo, TheCoolGameDev, jinjk, MillerHollinger</a></iframe></p>

<p align=middle>
  <h2 id="deadweight" align=middle><i>Dead Weight</i></h2>
</p>
<p align=middle>
  <img src="../deadweight.png" width=500>
</p>
<p align=middle>
Dead Weight is top-down dungeon crawler game developed in Unity. Fight Egyptian themed enemies to stop your soul from being fed to a bird. The game uses a modified version of the unity shadow caster for lighting.
</p>
<p align=middle>
<iframe frameborder="0" src="https://itch.io/embed/2904360" width="552" height="167"><a href="https://pragma-twice.itch.io/dead-weight">Dead Weight by Pragma-Twice, sword0fkas, nikkudo, jinjk</a></iframe>
  
<p align=middle>
  <h2 id="anderson" align=middle><i>Home for Anderson</i></h2>
</p>
<p align=middle>
  <img src="../gameplay.png" width=500>
</p>
<p align=middle>
Home for Anderson is a puzzle game made in Unity. Control multiple gems as you use their unique abilies to traverse the cave. The game uses the Unity NavMesh system for movement and a custom collision system for a water-freezing mechanic.
</p>
<p align=middle>
<iframe frameborder="0" src="https://itch.io/embed/2390986" width="552" height="167"><a href="https://pragma-twice.itch.io/homeforanderson">Home for Anderson by Pragma-Twice</a></iframe>
</p>

<p align=middle>
  <h2 id="aetherborne" align=middle><i>Aetherborne</i></h2>
</p>
<p align=middle>
  <img src="../atherborne.png" width=500>
</p>
<p align=middle>
Aetherborne is an online trading card game made in Unity. Aetherborne has a unique hexgrid system for placing and moving creatures. The game use Mirror as a backend for networking and has an elegant system for creating new cards.
</p>

<p align=middle>
  <h2 id="sanchari" align=middle><i>Sanchari</i></h2>
</p>
<p align=middle>
  <img src="../gameplay3.png" width=500>
</p>
<p align=middle>
Sanchari is a rythem game based on Indian mythology. The game features a system for creating song maps. 
</p>
<p align=middle>
<iframe frameborder="0" src="https://itch.io/embed/2608498" width="552" height="167"><a href="https://pragma-twice.itch.io/sanchari">Sanchari by Pragma-Twice, jinjk, sword0fkas</a></iframe>

<p align=middle>
  <h2 id="pipe" align=middle><i>Pipe Game</i></h2>
</p>
<p align=middle>
  <img src="../pipegame.png" width=500>
</p>
<p align=middle>
Pipe Game is a small highscore-based game made in unity for a one week game jam. Try to pipe the steam to the houses that need it before they blow up. Pipe Game features the unity particles system.
</p>
<p align=middle>
<a href="https://github.com/Clayton-Toste/PipeGame">Github</a>
</p>

<br>
<p align=middle>
  <h2 id="jello" align=middle><i>CS184 Final: Jello</i></h2>
</p>
<p align=middle>
  <img src="../jello.gif" width=400>
</p>
<p align=middle>
Jello is a program made for CS184 that simulates jellow physics with crosshatching shading. Runs on the GPU using Vulkan for increased performance.
</p>
<p align=middle>
<a href="https://github.com/Clayton-Toste/CS184-final-jello/tree/master">Github</a>
</p>

<br>
<p align=middle>
  <h2 id="deblurred" align=middle><i>Deblurred Diffusion</i></h2>
</p>
<p align=middle>
  <img src="../deblur.png" width=600>
</p>
<p align=middle>
Deblurred Diffusion is a technique I invented to improve current diffusion networks while working in the Gordon Lab at Stanford. It works by applying a filter to the noise in latent space to create a less blurry image
</p>

<br>
<p align=middle>
  <h2 id="shape" align=middle><i>Shape-Guided Diffusion with Inside-Outside Attention</i></h2>
</p>
<p align=middle>
  <img src="../SGD.png" width=600>
</p>
<p align=middle>
Shape-Guided Diffusion is a paper I co-published as third author with Seth Park in Professor Trevor Darrell's lab. The paper proposes a new method for object replacement that considers the shape of the original object.
</p>
<p align=middle>
<a href="https://shape-guided-diffusion.github.io/">Website</a>
</p>

<br>
<p align=middle>
  <h2 id="sa" align=middle><i>Surface Analysis</i></h2>
</p>
<p align=middle>
  <img src="../SA.png" width=400>
</p>
<p align=middle>
Surface Analysis is a program I made in C++ that interfaces with PyMol's python API to analyze patchs of hydrophobicity and charge of the surface of proteins. Created for Ting Xu Lab.
</p>
<p align=middle>
<a href="https://github.com/Clayton-Toste/Surface-Anaysis/tree/master">Github</a>
</p>

<br>
<p align=middle>
  <h2 id="mole" align=middle><i>Mole Game</i></h2>
</p>
<p align=middle>
  <img src="../molegame.png" width=200>
</p>
<p align=middle>
Mole Game is a small game I made using SDL to celebrate Mole day. Dig down to collect protons and neutrons to build heavier isotopes while avoiding darkmatter.
</p>
<p align=middle>
<a href="https://github.com/Clayton-Toste/Mole/tree/master">Github</a>
</p>
</div>
