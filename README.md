# Brick-Breaker
Created in JavaScript canvas that takes users house to input the control paddle. The user is required to break all bricks in-order to win

<!DOCTYPE html>
<html>
	<head>
		<meta charset = "utf-8">
		<title>Assignment 1 Part 4</title>
	</head>
	<body>
		<canvas id="gameCanvas" width="800" height="500"></canvas>
		<script>

			//Global variables for the ball
			var ballPosX = 300;
			var ballPosY = 400;
			var ballSpeedX = 7;
			var ballSpeedY = 5;

			//Global variables for the bricks
			var B_W = 115;
			var B_H = 20;
			var brickSpacing = 2;
			var brickColmns = 7;
			var brickRows = 4;
			var brickGrid = new Array(brickColmns * brickRows);
			var randomBrickColour = 1;
			var colour1 = getRandomColor();
			var colour2 = getRandomColor();
			var colour3 = getRandomColor();
			var colour4 = getRandomColor();

			//Global variables for the bar
			var barHeight = 100;
			var barThickness = 10;
			var barDist = 300;
			var paddle1X = 250;

			var canvas
			var canvasContext;

			//global variable for calculating mouse position
			var mouseX = 0;
			var mouseY = 0;

			//global variables for the lives and score 
			var lives = 3;
			var score = 0;
			//global variabels for the win/lose screens 
			var displayWinScreen = false;
			var displayLoseScreen = false;

			//function for what happens when the page loads
			window.onload = function() {
				canvas = document.getElementById('gameCanvas');
				canvasContext = canvas.getContext('2d');

				//sets the frame rate for the website
				var fps = 30;
				//sets the refresh interval for the website and how fast the objects get refreshed 
				setInterval(refresh, 1000/fps);

				canvas.addEventListener('mousemove', updateMousePos);
				canvas.addEventListener('mousedown', handleMouseClick);

				resetBrick();
			}
			//calls the movement of the objects function and redisplays the objects 
			function refresh() {
				movement();
				display();
			}
			//function updates the paddle based on the movement of the mouse 
			function updateMousePos(evt) {
				var rect = canvas.getBoundingClientRect();
				var root = document.documentElement;

				mouseX = evt.clientX - rect.left - root.scrollLeft;
				mouseY = evt.clientY - rect.top - root.scrollTop;

				paddle1X = mouseX - barHeight/2;
			}

			//mouse click is only used when you either win screen is show and lose screen is shown to make them go away and reset the game.
			function handleMouseClick(evt) {
    			if(displayWinScreen) {
    				score = 0;
    				displayWinScreen = false;
    			}
    			if(displayLoseScreen) {
    				lives = 3;
    				displayLoseScreen = false;
    			}
    		}
    		//resets the bricks array to be loaded with a value and a new colour 
			function resetBrick() {
				for(var i=0; i<brickColmns * brickRows; i++) {
					randomBrickColour = Math.floor((Math.random() * 4)+1);
					brickGrid[i] = randomBrickColour;
				}
			}
			//resets the ball position and also resets the score and lives when the screens are shown 
			function resetBall() {
				lives = lives-1;
				if (lives == 0 || score == 28){
					displayLoseScreen = true;
					score =0;
					lives = 5;
					resetBrick();
				}
				ballPosX = canvas.width/2;
				ballPosY = canvas.height/2;
				ballSpeedX = 7;
				ballSpeedY = 5;
			}

			//main function to calculate movement 
			function movement() {
				if(displayWinScreen) { //doesnt show anything if win screen is shown
				return;
				}
		    
		    	if(displayLoseScreen) {//doesnt show anything if lose screen is shown
					return;
				}
				ballPosX += ballSpeedX; //incriments the ball  postion in the X direction
				ballPosY += ballSpeedY; //incriments the ball  postion in the Y direction

				//checks if the ball hit the ground, if it does then reset the ball else revese the ball movement 	
				if(ballPosY > canvas.height) {
					if(ballPosX > paddle1X &&
					   ballPosX < paddle1X + barThickness) {
						ballSpeedY = -ballSpeedY;
						
						var changeSpeedX = ballPosX
							-(paddle1X+barThickness/2);
						ballSpeedX = changeSpeedX * 0.01;
					} else {
						resetBall();
					}
				}
				
				//if the ball hits the wall 
				if(ballPosX > canvas.width) {
					ballSpeedX = -ballSpeedX;

				}
				//if the ball hits the wall 
				if(ballPosX < 0) {
					ballSpeedX = -ballSpeedX;

				}
			    //if the ball hits the wall 
			    if(ballPosY < 0){
			        ballSpeedY = -ballSpeedY;
			    }


				var CollisionCol = Math.floor(ballPosX / B_W);
				var CollisionRow = Math.floor(ballPosY / B_H);
				var brickIndexUnderBall = calcArray(CollisionCol, CollisionRow);

				if(CollisionCol >= 0 && CollisionCol < brickColmns &&
					CollisionRow >= 0 && CollisionRow < brickRows) {

					if(brickGrid[brickIndexUnderBall]) {
						brickGrid[brickIndexUnderBall] = false;
						ballSpeedY *= -1;
						score ++;
						if(score == 28){
							displayWinScreen = true;
							resetBall();
						}
					}
				
				}
				//calculates the location of the paddle to determine how the ball will move 
				var paddleTopEdgeY = canvas.height-barThickness;
				var paddleBottomEdgeY = paddleTopEdgeY + barDist;
				var paddleLeftEdgeX = paddle1X;
				var paddleRightEdgeX = paddleLeftEdgeX + barHeight;
				//if all the conditions are met then the ball will move back up towards the bricks
				if( ballPosY >= paddleTopEdgeY && // below the top of paddle
					ballPosY <= paddleBottomEdgeY && // above bottom of paddle
					ballPosX >= paddleLeftEdgeX && // right of the left side of paddle
					ballPosX <= paddleRightEdgeX) { // left of the left side of paddle
					
					ballSpeedY *= -1;
				}
			}
			    
			//used to calcualte the values in the array
			function calcArray(col, row) {
				return col + brickColmns * row;
			}
			//function draws the bricks
			function drawBricks() {

				for(var x=0;x<brickRows;x++) {
					for(var y=0;y<brickColmns;y++) {
						//goes through and uses the global values to draw the rectangeles and where they should go. 
						var arrayIndex = calcArray(y, x); 

						if(brickGrid[arrayIndex] ==1) {
							drawRect(B_W*y,B_H*x,
								B_W-brickSpacing,B_H-brickSpacing, colour1);
						} 
						if(brickGrid[arrayIndex] ==2) {
							drawRect(B_W*y,B_H*x,
								B_W-brickSpacing,B_H-brickSpacing, colour2);
						} 
						if(brickGrid[arrayIndex] ==3) {
							drawRect(B_W*y,B_H*x,
								B_W-brickSpacing,B_H-brickSpacing, colour3);
						} 
						if(brickGrid[arrayIndex] ==4) {
							drawRect(B_W*y,B_H*x,
								B_W-brickSpacing,B_H-brickSpacing, colour4);
						} 
					}
				} 

			} 
			//how everything should be displayed 
			function display() {
				drawRect(0,0, canvas.width,canvas.height, 'black'); // clear screen
				if(displayWinScreen) { //how the win screen looks
					canvasContext.font="20px Helvetica";
					canvasContext.fillStyle = 'white';
					canvasContext.fillText("You Win", 370, 200);
					canvasContext.fillText("Click to Restart Game", 340, 300);
					return;
				}
				if(displayLoseScreen) {//how the lose screen looks
				    canvasContext.font="20px Helvetica";
					canvasContext.fillStyle = 'white';
				    canvasContext.fillText("You Lose",370, 200);
				    canvasContext.fillText("Click to Restart Game", 340, 300);
					return;
				}

				drawCircle(ballPosX,ballPosY, 10, 'white'); // draw ball

				drawRect(paddle1X,canvas.height-barThickness , barHeight, barThickness, 'white'); //paddle


				drawBricks();

				canvasContext.fillStyle = 'white';

				canvasContext.font="15px Helvetica";
				canvasContext.fillText("Lives:", 700, 250);
				canvasContext.fillText(lives, 750, 250);

				canvasContext.fillText("Score:", 700, 280);
				canvasContext.fillText(score, 750, 280);
			}
			//function to draw the bricks
			function drawRect(topLeftX,topLeftY, boxWidth,boxHeight, fillColor) {
				canvasContext.fillStyle = fillColor;
				canvasContext.fillRect(topLeftX,topLeftY, boxWidth,boxHeight);
			}
			//function to draw the ball
			function drawCircle(centerX,centerY, radius, fillColor) {
				canvasContext.fillStyle = fillColor;
				canvasContext.beginPath();
				canvasContext.arc(centerX,centerY, 10, 0,Math.PI*2, true);
				canvasContext.fill();
			}
			//this function generates random colours for the bricks
			function getRandomColor() {
		    var letters = '0123456789ABCDEF';
		    var colour = '#';
		    for (var i = 0; i < 6; i++ ) {
		        colour += letters[Math.floor(Math.random() * 16)];
		    }
		    return colour;
			}

		</script>
	</body>
</html>
