(function(){
	$(document).ready(function(){
	    var game ={};
		
		game.stars =[];
		
		game.width = 550;
		game.height = 600;
		
		game.keys = [];
		game.projectiles = [];
		game.enemies = [];
		
		game.images = [];
		game.doneImages = 0;
		game.requiredImages = 0;
		
		game.gameOver = false; 
		game.gameWon = false;
		
		game.count = 24;
		game.division = 48;
		game.left = false;
		game.enemySpeed = 2;
		
		game.moving =false;
		
		game.fullShootTimer = 20;
		game.shootTimer = game.fullShootTimer;
	    
		game.player = {
			x:game.width / 2 - 50,
			y:game.height - 100,
			width: 100,
			height: 100,
			speed: 3,
			rendered: false
		};  
		
		game.contextBackground = document.getElementById("backgroundCanvas").getContext("2d");
		game.contextPlayer = document.getElementById("playerCanvas").getContext("2d");
	    game.contextEnemy = document.getElementById("enemyCanvas").getContext("2d");
		 $(document).keydown(function(e){
			 game.keys[e.keyCode ? e.keyCode : e.which] = true;
		 });
		  $(document).keyup(function(e){
			delete game.keys[e.keyCode ? e.keyCode : e.which];
		 });
		 function addBullet(){
			 game.projectiles.push({
				x: game.player.x + 41,
                y: game.player.y,
                size: 10,
                image: 2				
			 });
		 }
		
	    function init(){
			for(i = 0; i < 600; i++){
				game.stars.push({
				    x: Math.floor(Math.random() * game.width),
				    y: Math.floor(Math.random() * game.height),
				    size: Math.random() * 5,
				//	speed: Math.random() * 11 
				});
			}
			for(y = 0; y <5; y++){
				for(x = 0; x < 6; x++){
					game.enemies.push({
						x: (x * 70) + 100,
						y: y * 70,
						width: 35,
						height: 35,
						image: 1,
						dead: false,
						deadTime: 25
					});
				}
			}
			loop();
			setTimeout(function(){
				game.moving = true;
			},9990);
			
		}
		
			
		function addStars(num){
			for(i = 0; i < num; i++){
				game.stars.push({
				    x: Math.floor(Math.random() * game.width),
				    y: game.height + 10,
				    size: Math.random() * 5,
					//speed: Math.random() * (2-1) + 1
				});
			}
		}
		
		function update(){
			addStars(1);
			game.count++;
			if(game.shootTimer > 0)game.shootTimer --;
			//console.log(game.shootTimer);
			for(i in game.stars){
	         //game.stars[i].y+= game.stars[i].speed;      not working but im trying to acces each star in the array and give it a random speed
				if(game.stars[i].y <= -5){
					game.stars.splice(i, 1);
				
				}
				game.stars[i].y-=Math.random();
				
			}
			
			if(game.keys[37] || game.keys[65]){
				if (game.player.x > 0){
					game.player.x-=game.player.speed;
					game.player.rendered = false;
				}
			}
			if(game.keys[39] || game.keys[68]){
				if(game.player.x <=game.width-game.player.width){
				game.player.x+=game.player.speed;
				game.player.rendered = false;
				}
			}
			if(game.keys[38] || game.keys[87]){
				if (game.player.y > 0){
					game.player.y-=game.player.speed;
					game.player.rendered = false;
				}
			}
			if(game.keys[40] || game.keys[83]){
				if(game.player.y <=game.height-game.player.height){
				game.player.y+=game.player.speed;
				game.player.rendered = false;
				}
				
			}
			if(game.count % game.division == 0){
				game.left = !game.left;
			}
			for(i in game.enemies){
				if(!game.moving){
				if(game.left){
					game.enemies[i].x -= game.enemySpeed;
				  }else{
				  game.enemies[i].x += game.enemySpeed;
				  }
				}if(game.moving){
					game.enemies[i].y++;
				}
				if(game.enemies[i].y>= game.height){
					game.gameOver = true;
					//console.log("loser")
				}
				
			}
			for(i in game.projectiles){
				game.projectiles[i].y -=3;
				if(game.projectiles[i].y <= game.projectiles[i].size){
					game.projectiles.splice(i, 1);
				}
			}
			if(game.keys[32] && game.shootTimer <= 0){
				addBullet();
				game.shootTimer = game.fullShootTimer;
			}
			for(m in game.enemies){
				for(p in game.projectiles){
					if(collision(game.enemies[m], game.projectiles[p])){
					 game.enemies[m].dead = true;
					 game.enemies[m].image = 3;
					 game.contextEnemy.clearRect(game.projectiles[p].x - 1,game.projectiles[p].y - 1,game.projectiles[p].size + 2,game.projectiles[p].size + 5);
					 game.projectiles.splice(p, 1);
					}
					
				}
			}
			for(i in game.enemies){
				if(game.enemies[i].dead){
					game.enemies[i].deadTime--;
				}
				if(game.enemies[i].dead && game.enemies[i].deadTime <= 0){
					game.contextEnemy.clearRect(game.enemies[i].x -2, game.enemies[i].y, game.enemies[i].width + 5, game.enemies[i].height + 5);
					game.enemies.splice(i, 1);
				}
			}
			if(game.enemies.length <= 0){
				game.gameWon = true;
				
			}
		}
		
		function render(){
			game.contextBackground.clearRect(0, 0, game.width, game.height);
			game.contextBackground.fillStyle = "white";
			for(i in game.stars){
				var star = game.stars[i];
				game.contextBackground.fillRect(star.x, star.y, star.size, star.size);
			}
			if(!game.player.rendered){
			game.contextPlayer.clearRect(game.player.x,game.player.y-10, game.player.width * 2, game.player.height * 2 );
			game.contextPlayer.drawImage(game.images[0], game.player.x, game.player.y, game.player.width, game.player.height);
			game.player.rendered = true;
			}
			for(i in game.enemies){
				var enemy = game.enemies[i];
				game.contextEnemy.clearRect(enemy.x - 2, enemy.y-2, enemy.width + 4,enemy.height + 4);
				game.contextEnemy.drawImage(game.images[enemy.image], enemy.x, enemy.y,enemy.width,enemy.height);
			}
             for(i in game.projectiles){
				 var proj = game.projectiles[i];
				 game.contextEnemy.clearRect(proj.x, proj.y, proj.size, proj.size + 3);
				 game.contextEnemy.drawImage(game.images[proj.image], proj.x, proj.y, proj.size, proj.size);
			 }
             if(game.gameOver){
				 game.contextBackground.font ="bold 50px monoco";
		         game.contextBackground.fillStyle ="white";
		         game.contextBackground.fillText ("game over !!",game.width / 2 -100, game.height / 2 - 25);
			 }
			   if(game.gameWon){
				 game.contextBackground.font ="bold 50px monoco";
		         game.contextBackground.fillStyle ="white";
		         game.contextBackground.fillText ("allright !!",game.width / 2 -100, game.height / 2 - 25);
			 }
         			 
		
		}   
	      	
		
		function loop(){
			requestAnimFrame(function(){
				loop();
			});
			update();
			render();
			
		}
		function initImages(paths){
			game.requiredImages = paths.length;
			for(i in paths){
				var img = new Image;
				img.src = paths[i];
				game.images[i] = img;
				game.images[i].onload =function(){
					game.doneImages++;
				}
			}
		}
		function collision(first, second){
			return !(first.x > second.x + second.width ||
			    first.x + first.width < second.x ||
				first.y > second.y + second.height ||
				first.y + first.height < second.y);
		}
		
		function checkImages(){
			if(game.doneImages >=game.requiredImages){
				init();
			}else{
				setTimeout(function(){
				checkImages();
				}, 1);
			}
		}
		game.contextBackground.font ="bold 50px monoco";
		game.contextBackground.fillStyle ="white";
		game.contextBackground.fillText ("loading",game.width / 2 -100, game.height / 2 - 25);
		initImages(["agian.png","enemy.png","bullet.png","explsn.png"]);
		checkImages();
         		
	});
})();	

window.requestAnimFrame = (function(){
	return window.requestAnimationFrame        ||
	       window.webkitRequestAnimationFrame  ||
		    window.mozRequestAnimationFrame    ||
			 window.oRequestAnimationFrame||
		    window.msRequestAnimationFrame    ||
			function(callback){
				window.setTimeOut(callback, 1000 / 60);
			};
				
})();
