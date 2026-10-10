#S1 bird scn
extends CharacterBody2D

const GRAVITY : int = 1000
const MAX_VEL : int = 600
const FLAP_SPEED : int = -500
var flying : bool = false
var falling : bool = false
const START_POS = Vector2(100,  400)

func _ready():
	reset()

func reset():
	falling = false
	flying = false
	position = START_POS
	set_rotation(0)
	
func _physics_process(delta):
	if flying or falling:
		velocity.y += GRAVITY * delta
	
		if velocity.y > MAX_VEL:
			velocity.y = MAX_VEL
	
		if flying:
			set_rotation(deg_to_rad(velocity.y * 0.05))
			$AnimatedSprite2D.play()
	
		elif falling:
			set_rotation(PI/2)
			$AnimatedSprite2D.stop()
		move_and_collide(velocity * delta)
	else:
		$AnimatedSprite2D.stop()
	
func flap():
	velocity.y = FLAP_SPEED


#S2 Main scn
extends Node

var game_running : bool
var game_over : bool
var scroll : int
var score : int
const SCROLL_SPEED : int = 4
var screen_size : Vector2i
var ground_height : int
var pipes : Array
const PIPE_DELAY : int = 100
const PIPE_RANGE : int = 200

func _ready():
	new_game()

func new_game():
	# reset variables
	game_running = false
	game_over = false
	score = 0
	scroll = 0
	$Bird.reset()

func _input(event):
	if game_over == false:
		if event is InputEventMouseButton:
			if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
				if game_running == false:
					start_game()
				else:
					if $Bird.flying:
						$Bird.flap()

func start_game():
	game_running = true
	$Bird.flying = true
	$Bird.flap()


#S3 Ground Scn
extends Area2D

signal hit

func _on_body_entered(body):
	hit.emit()



#edit main ready and process
func _ready():
	screen_size = get_window().size
	new_game()


func _process(delta):
	if game_running:
			scroll += SCROLL_SPEED
			if scroll > screen_size.x:
				scroll = 0
			$Ground.position.x = -scroll

#S4 pipe scene
extends Area2D

signal hit


func _on_body_entered(body):
	hit.emit()


#edit the main add export, update ready, add timer, add generate pipes, update new game, add bird hit, update process, update start game
@export var pipe_scene : PackedScene

func _ready():
	screen_size = get_window().size
	ground_height = $Ground.get_node("Sprite2D").texture.get_height()
	new_game()

func _on_pipe_timer_timeout():
	generate_pipes()
	
func generate_pipes():
	var pipe = pipe_scene.instantiate()
	pipe.position.x = screen_size.x + PIPE_DELAY
	pipe.position.y = (screen_size.y - ground_height) / 2 + randi_range(-PIPE_RANGE, PIPE_RANGE)
	pipe.hit.connect(bird_hit)
	add_child(pipe)
	pipes.append(pipe)

func new_game():
	game_running = false
	game_over = false
	score = 0
	scroll = 0
	pipes.clear()
	generate_pipes()
	$Bird.reset()

func _process(delta):
	if game_running:
		scroll += SCROLL_SPEED
		if scroll > screen_size.x:
			scroll = 0
		$Ground.position.x = -scroll
		for pipe in pipes:
			pipe.position.x -= SCROLL_SPEED

func start_game():
	game_running = true
	$Bird.flying = true
	$Bird.flap()
	$PipeTimer.start()


#S5 Collision add check top, add stop game, update input, update bird hit, add ground hit
func check_top():
	if $Bird.position.y < 0:
		$Bird.falling = true
		stop_game()

func stop_game():
	$PipeTimer.stop()
	$Bird.flying = false
	game_running = false
	game_over = true

func bird_hit():
	$Bird.falling = true
	stop_game()

func _on_ground_hit():
	$Bird.falling = false
	stop_game()


#S6 add score signal in pipes
func _on_score_area_body_entered(body):
	scored.emit()

#update main generate pipes, add func scored, update new game
func generate_pipes():
	var pipe = pipe_scene.instantiate()
	pipe.position.x = screen_size.x + PIPE_DELAY
	pipe.position.y = (screen_size.y - ground_height) / 2 + randi_range(-PIPE_RANGE, PIPE_RANGE)
	pipe.hit.connect(bird_hit)
	pipe.scored.connect(scored)
	add_child(pipe)
	pipes.append(pipe)

func scored():
	score += 1
	$ScoreLabel.text = "SCORE: " + str(score)

#S7 add game over script
extends Button

signal restart

func _on_pressed():
	restart.emit

#update main new game, stop game, link restart
func new_game():
	game_running = false
	game_over = false
	score = 0
	scroll = 0
	$ScoreLabel.text = "SCORE: " + str(score)
	$GameOver.hide()
	pipes.clear()
	generate_pipes()
	$Bird.reset()

func stop_game():
	$PipeTimer.stop()
	$GameOver.show()
	$Bird.flying = false
	game_running = false
	game_over = true

func _on_game_over_restart():
	new_game()

#update new game with pipe group
func new_game():
	game_running = false
	game_over = false
	score = 0
	scroll = 0
	$ScoreLabel.text = "SCORE: " + str(score)
	$GameOver.hide()
	get_tree().call_group("pipes", "queue_free")
	pipes.clear()
	generate_pipes()
	$Bird.reset()
