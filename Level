package com.me.catquest;


import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input;
import com.badlogic.gdx.Screen;
import com.badlogic.gdx.Input.Keys;
import com.badlogic.gdx.graphics.Color;
import com.badlogic.gdx.graphics.GL10;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.BitmapFont;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.maps.tiled.TiledMap;
import com.badlogic.gdx.maps.tiled.TiledMapTileLayer;
import com.badlogic.gdx.maps.tiled.TmxMapLoader;
import com.badlogic.gdx.maps.tiled.TiledMapTileLayer.Cell;
import com.badlogic.gdx.maps.tiled.renderers.OrthogonalTiledMapRenderer;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.scenes.scene2d.Stage;
import com.badlogic.gdx.scenes.scene2d.ui.Image;
import com.badlogic.gdx.scenes.scene2d.ui.Label;
import com.badlogic.gdx.scenes.scene2d.ui.Label.LabelStyle;
import com.badlogic.gdx.scenes.scene2d.utils.Align;
import com.badlogic.gdx.utils.Array;
import com.badlogic.gdx.utils.Pool;


public abstract class Level implements Screen{

	int maxPosX;
	int maxPosY;
	Block block;
	SpriteBatch spriteBatch;
	SpriteBatch batch;
	BitmapFont white;
	Label label;
	Label label2;
	private TiledMap map;
	protected OrthogonalTiledMapRenderer renderer;
	private OrthographicCamera camera;
	Stage stage;
	Stage stage2;
	Image imageBlock;
	Image imageCat;
	Player player;
	int spawn[];
	boolean pause = false;
	private Pool<Rectangle> rectPool = new Pool<Rectangle>(){
		@Override
		protected Rectangle newObject(){
			return new Rectangle();
		}
	};
	int pauseMenuPosition;
	LabelStyle blackstyle;
	LabelStyle style;
	Label pauseLabel1;
	Label pauseLabel2;
	int maxMenuPosition;
	String levelName;
	boolean cheatsOn = false;

	

	
	private Array<Rectangle> tiles = new Array<Rectangle>();
	private Array<Rectangle> exit = new Array<Rectangle>();
	

	
	Level(int maxX, int maxY, String level, int[] s){
		spriteBatch = new SpriteBatch();
		maxPosX = maxX;
		maxPosY = maxY;
		levelName = level;
		map = new TmxMapLoader().load("data/" + levelName + ".tmx");
		block = new Block();
		spawn = s;
	}
	
	@Override
	public void show() {

		//load the map, set the unit scale to 1/32 (1 unit = 32 pixels)
		renderer = new OrthogonalTiledMapRenderer(map, 1 / 32f);

		//create a new camera, shows us 20,20 view of world
		camera = new OrthographicCamera();
		camera.setToOrtho(false, 20, 20);
		camera.update();
		
		//creates the player we want to move around the world
		//and sets position to 10, 10
		player = new Player();
		player.pos.set(spawn[0], spawn[1]);
		
		
		//Prepares map and adds enemies
		map = block.prepareMap(map, maxPosX, maxPosY);
		addEnemies();
		
		//Prepares batch for UI elements
		batch = new SpriteBatch();
		white = new BitmapFont(Gdx.files.internal("data/whitefont.fnt"), false);
		
		blackstyle = new LabelStyle(white, Color.BLACK);
		style = new LabelStyle(white, Color.WHITE);
		
		maxMenuPosition = 2;
		pauseMenuPosition = 1;
	}
	
	@Override
	public void render(float delta) {
		//clears screen
		Gdx.gl.glClearColor(0.7f, 0.7f, 1.0f, 1);
		Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);
		
		if(pause)
			pauseMenu(delta);
		else
			updatePlayer(delta);//updates player using deltatime
		
		// set the tile map renderer view based on what the
		// camera sees and render the map
		renderer.setView(camera);
		renderer.render();
		
		//renders player and enemies
		player.renderCharacter(renderer);
		renderEnemies();
		

		//adjusts label for new block value and then renders stage for UI elements
		label.setText(Integer.toString(player.blocks));
		label2.setText("x" + Integer.toString(player.lives));
		stage.act(delta);
		if(pause)
			stage2.act(delta);
		batch.begin();
		stage.draw();
		if(pause)
			stage2.draw();
		batch.end();
	}
	
	private void pauseMenu(float delta){
		if (Gdx.input.isKeyPressed(Keys.W) || Gdx.input.isKeyPressed(Keys.UP)) { 
			if(pauseMenuPosition != 1);	
				pauseMenuPosition = 1;
				pauseLabel1.setStyle(style);
				pauseLabel2.setStyle(blackstyle);
		}
		if (Gdx.input.isKeyPressed(Keys.S) || Gdx.input.isKeyPressed(Keys.DOWN)) { 
			if(pauseMenuPosition != maxMenuPosition);	
				pauseMenuPosition = 2;
				pauseLabel1.setStyle(blackstyle);
				pauseLabel2.setStyle(style);
		}
		if (Gdx.input.isKeyPressed(Keys.SPACE)||Gdx.input.isKeyPressed(Keys.ENTER)) { 
			if(pauseMenuPosition == 1)
				pause = false;
			if(pauseMenuPosition == 2){
				pause = false;
				restart();
			}
		}
		if(Gdx.input.isKeyPressed(Keys.C)&& Gdx.input.isKeyPressed(Keys.A) && Gdx.input.isKeyPressed(Keys.T))
		{
			cheatsOn = true;
			Audio.stopMusic();
			Audio.playMusic(1);
		}
		
		if(Gdx.input.isKeyPressed(Keys.N) && Gdx.input.isKeyPressed(Keys.E) && Gdx.input.isKeyPressed(Keys.X) && Gdx.input.isKeyPressed(Keys.T)){
			pause = false;
			load_next_level();
		}
		
		if(Gdx.input.isKeyPressed(Keys.B) && Gdx.input.isKeyPressed(Keys.A) && Gdx.input.isKeyPressed(Keys.C) && Gdx.input.isKeyPressed(Keys.K)){
			pause = false;
			load_previous_level();
		}	
		
	}
	
	private void restart(){
		player.vel.x = 0;
		player.vel.y = 0;
		player.pos.set(spawn[0], spawn[1]);
		TiledMap newMap = new TmxMapLoader().load("data/" + levelName + ".tmx");
		map = block.prepareMap(newMap, maxPosX, maxPosY);
		renderer.setMap(map);
		addEnemies();
		player.blocks = 0;
		player.lives -= 1;
		
	}
	
	protected boolean enemyColision(Enemy enemy){
		boolean returnValue = false;
		Rectangle playerRect = rectPool.obtain();
		Rectangle enemyRect = rectPool.obtain();
		playerRect.set(player.pos.x, player.pos.y, player.WIDTH, player.HEIGHT);
		enemyRect.set(enemy.pos.x, enemy.pos.y, enemy.WIDTH, enemy.HEIGHT);
		if(playerRect.overlaps(enemyRect)){
			if(player.vel.y < 0)
				returnValue = true;
			else{
				returnValue = false;
				player.death(spawn);
			}
		}
		rectPool.free(enemyRect);
		rectPool.free(playerRect);
		return returnValue;
			
	}
	
	protected Character colisionDetection(Character character){
		//perform collision detection & response, on each axis, separately
		character = colisionDetectionX(character);
		character = colisionDetectionY(character);
		return character;
	}
	
	private boolean exitCheck(){
		Rectangle rect = rectPool.obtain();
		rect.set(player.pos.x, player.pos.y, player.WIDTH, player.HEIGHT);
		int startX, endX, startY, endY;
		startX = endX = (int) (player.pos.x + player.WIDTH + player.vel.x);  
		startY = (int) (player.pos.y);
		endY = (int) (player.pos.y + player.HEIGHT);
		getTiles(startX, startY, endX, endY, exit, 3);
		for(Rectangle tile: exit){
			if(rect.overlaps(tile))
				return true;
		}
		return false;
	}
	
	private Character colisionDetectionX(Character character){
		// if the player is moving right, check the tiles to the right of it's
		// right bounding box edge, otherwise check the ones to the left
		Rectangle rect = rectPool.obtain();
		rect.set(character.pos.x, character.pos.y, character.WIDTH, character.HEIGHT);
		int startX, startY, endX, endY;  
		if (character.vel.x > 0){
			startX = endX = (int) (character.pos.x + character.WIDTH + character.vel.x);  
		} 
		else{
			startX = endX = (int) (character.pos.x + character.vel.x); 
		}
		startY = (int) (character.pos.y);
		endY = (int) (character.pos.y + character.HEIGHT);
		getTiles(startX, startY, endX, endY, tiles, 1);  
		rect.x += player.vel.x;
		for (Rectangle tile : tiles){
			if (rect.overlaps(tile)){
				character.vel.x = 0;  
				break;
			}
		}
		rectPool.free(rect);
		return character;
	}
	
	private Character colisionDetectionY(Character character){
		Rectangle rect = rectPool.obtain();
		rect.set(character.pos.x, character.pos.y, character.WIDTH, character.HEIGHT);
		int startX, startY, endX, endY;  
		// if the player is moving upwards, check the tiles to the top of it's
		// top bounding box edge, otherwise check the ones to the bottom
		if (character.vel.y > 0){
			startY = endY = (int) (character.pos.y + character.HEIGHT + character.vel.y);
		} 
		else{
			startY = endY = (int) (character.pos.y + character.vel.y);
		}
		startX = (int) (character.pos.x);
		endX = (int) (character.pos.x + character.WIDTH);

		getTiles(startX, startY, endX, endY, tiles, 1);
		rect.y += character.vel.y;
		for (Rectangle tile : tiles){
			if (rect.overlaps(tile)){
				//We reset the player y-position here  so it is just below/above 
				//the tile we collided with. This removes bouncing
				if(character.vel.y > 0){
					character.pos.y = tile.y - character.HEIGHT;
					} 
				else{
					character.pos.y = tile.y + tile.height;
					//if we hit the ground, mark us as grounded so we can jump again
					character.grounded = true;
				}
				character.vel.y = 0;
				break;
			}
		}
		rectPool.free(rect);
		return character;
	}
	
	
	private void updatePlayer(float deltaTime) {
		if(player.lives==0)
			gameOver();

		//if no time has passed since last use, does not update
		// and adjusts player stateTime for animations
		if (deltaTime == 0)
			return;
		player.stateTime += deltaTime;
				
		//calculates current mouse location
		int xcoord = (int)((camera.position.x-camera.viewportWidth/2)+(Gdx.input.getX()/32));
		int ycoord = (int)((camera.position.y-camera.viewportHeight/2)+(camera.viewportHeight-Gdx.input.getY()/32)-1);
		
		//checks input and applies jump or movement velocity 
		//to player's velocity if input is found
		if ((Gdx.input.isKeyPressed(Keys.SPACE)||Gdx.input.isKeyPressed(Keys.W)) && player.grounded) { 
			player.jumpNoise.play(0.35f);
			player.vel.y += player.JUMP_VEL; 
			player.state = Player.State.Jump;   
			player.grounded = false;	
		}
		
		if ((Gdx.input.isKeyPressed(Keys.SPACE)||Gdx.input.isKeyPressed(Keys.W)) && !player.grounded && cheatsOn) { 
			if(player.vel.y < 0)
				player.vel.y = 0;
		}
		

		if (Gdx.input.isKeyPressed(Keys.LEFT) || Gdx.input.isKeyPressed(Keys.A)) {
			player.vel.x = -player.MAX_VEL;  
			if (player.grounded)  		
				player.state = Player.State.Walk;  
			player.facesRight = false;  
		}

		if (Gdx.input.isKeyPressed(Keys.RIGHT) || Gdx.input.isKeyPressed(Keys.D)){
			player.vel.x = player.MAX_VEL; 
			if (player.grounded)         
				player.state = Player.State.Walk;  
			player.facesRight = true;		
		}
		
		if (Gdx.input.isKeyPressed(Keys.ESCAPE)) {   
			pause = true;
		}
		
		if (Gdx.input.isButtonPressed(Input.Buttons.LEFT)){
			//if left mouse is clicked and you have blocks available
			//sends current mouse coordinates to placeBlocks method
			if((cheatsOn || player.blocks > 0) && block.canPlaceCheck(map, xcoord, ycoord)){
				map = block.placeBlocks(map, xcoord, ycoord);
				player.blocks--;
			}
		}
		
		if (Gdx.input.isButtonPressed(Input.Buttons.RIGHT)){
			//if right mouse is clicked and block is a placeable block
			//sends current mouse coordinates to removeBlocks method
			if(block.placeableBlockCheck(map, xcoord, ycoord, player.pos.x, player.pos.y) || cheatsOn)	{
				map = block.removeBlocks(map, xcoord, ycoord);
				player.blocks++;
			}
		}
		//applies gravity if we are falling
		player.vel.add(0, player.GRAVITY);
		
		//reduces players velocity to 0 if less than 1
		if (Math.abs(player.vel.x) < 1)  {
			player.vel.x = 0; 
			if (player.grounded)   
				player.state = Player.State.Stand; 
		}
		
		enemyAI1();
		
		//multiply by deltatime so we know how far to go in this frame
		player.vel.scl(deltaTime); 
		
		//runs player through colision detection
		colisionDetection(player);
		colision();
		
		// unscale the velocity by the inverse delta time and set
		// the latest position
		player.pos.add(player.vel);
		player.vel.scl(1 / deltaTime);
		
		enemyAI2();
		
		//checks if player is past min/max values, and postions them back if they are
		if(player.pos.x < 0)
			player.pos.x = 0;
		if(player.pos.x > maxPosX)
			player.pos.x = maxPosX;
		if(player.pos.y < 0)
			player.death(spawn);
		
		
		
			
		//adjusts camera for player's new location,  
		//only adjusts y position if player is above a certain height
		//only adjusts x position if player isn't near min/max X values
		{
		if(player.pos.x < camera.viewportWidth/2) 
			camera.position.x = camera.viewportWidth/2;
		else if(player.pos.x > maxPosX-(camera.viewportWidth/2)+1)
			camera.position.x = maxPosX-(camera.viewportWidth/2)+1;
		else
			camera.position.x = player.pos.x;}
		{
		if(player.pos.y < camera.viewportHeight/2) 
			camera.position.y = camera.viewportHeight/2;
		else if(player.pos.y > maxPosY-(camera.viewportHeight/2))
			camera.position.y = maxPosY-(camera.viewportHeight/2);
		else
			camera.position.y = player.pos.y;	}
		camera.update();
		
		//apply damping on x-axis so we don't walk infinitely when key is pressed
		player.vel.x *= Player.DAMP;
		
		//renders the transparent block under the mouse
		if(cheatsOn)
			map = block.renderOverlayBlock(map, 1, deltaTime, xcoord, ycoord);
		else 
			map = block.renderOverlayBlock(map, player.blocks, deltaTime, xcoord, ycoord);
		if(exitCheck()){
			load_next_level();
		}
	}

	private void getTiles(int startX, int startY, int endX, int endY,Array<Rectangle> array, int i){
		//gets layer i from map, places all rectangles from tiles back into pool, then clears tiles array
		TiledMapTileLayer layer = (TiledMapTileLayer) map.getLayers().get(i); 
		rectPool.freeAll(array);
		array.clear();
		
		//iterates across start/end y and start/end x
		//then sets cell object equal to cell located at x and y on map
		for (int y = startY; y <= endY; y++)  
		{
			for (int x = startX; x <= endX; x++)
			{
				Cell cell = layer.getCell(x, y);  
				//if cell isn't null, it takes a rectangle from the pool of rectangles and 
				//sets the rectangles coordinates to x and y, then adds the rectangle back to tiles array
				if (cell != null) 
				{
					Rectangle rect = rectPool.obtain(); 
					rect.set(x, y, 1, 1);  
					array.add(rect);
				}
			}
		}
	}

	protected void load_next_level(){
		
	}
	
protected void load_previous_level(){
		
	}
	
	private void gameOver(){
		GameController.game_over();
		this.dispose();
	}
	
	//mandatory override stuff for subclasses
	@Override
	public void resize(int width, int height) {
		
		//First checks if there's a stage, then makes one if there isn't
		if (stage == null)
			stage = new Stage(width, height, true);
		stage.clear();
		
		if(stage2 == null)
			stage2 = new Stage(width, height, true);
		stage2.clear();
		
		//Creates Label Style then the actual label
		label = new Label(Integer.toString(player.blocks), style);
		label.setWidth(64);
		label.setPosition(571,576);
		label.setAlignment(Align.center);
		
		label2 = new Label("x" + Integer.toString(player.lives), style);
		label2.setWidth(64);
		label2.setPosition(64,576);
		label2.setAlignment(Align.center);
		
		pauseLabel1 = new Label("Resume",style);
		pauseLabel1.setWidth(64);
		pauseLabel1.setPosition(290,300);
		pauseLabel1.setAlignment(Align.center);
		
		pauseLabel2 = new Label("Restart",blackstyle);
		pauseLabel2.setWidth(64);
		pauseLabel2.setPosition(290,200);
		pauseLabel2.setAlignment(Align.center);
		
		
		//creates an image of the block for the UI
		imageBlock = new Image(block.blockTexture);
		imageBlock.setHeight(50);
		imageBlock.setWidth(50);
		imageBlock.setPosition(576, 576);
		
		//creates an image of cathead for UI lives
		imageCat = new Image(new Texture(Gdx.files.internal("data/mario.png")));
		imageCat.setHeight(50);
		imageCat.setWidth(50);
		imageCat.setPosition(14, 576);
		
		//adds block and label to stage
		stage.addActor(imageBlock);
		stage.addActor(imageCat);
		stage.addActor(label);
		stage.addActor(label2);
		
		stage2.addActor(pauseLabel1);
		stage2.addActor(pauseLabel2);
	}
	protected void enemyAI1(){}
	protected void enemyAI2(){}
	protected void enemyColision(){}
	protected void addEnemies(){}
	protected void renderEnemies(){}
	@Override
	public void hide() {}
	@Override
	public void resume() {}
	@Override
	public void dispose() {	}
	@Override
	public void pause() {}
	protected void colision() {}
}
