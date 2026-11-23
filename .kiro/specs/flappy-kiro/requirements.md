# Requirements Document

## Introduction

Flappy Kiro is a retro, endless-scrolling game that runs in a web browser. The player guides a ghost character (Ghosty) through gaps between pairs of green pipes. The ghost falls under gravity and rises when the player triggers a jump. Each pipe pair passed scores one point. Ghosty has several Lives: a collision with a pipe costs one Life and starts a short Invincibility window, and the round ends when the Lives reach zero. Touching the ground or the ceiling ends the round immediately, regardless of the remaining Lives. The visual design follows `img/example-ui.png`: a light-blue, hand-sketched sky background with semi-transparent white rounded clouds that drift at different speeds to create a sense of depth, green pipes with darker caps (top pipes hang from the ceiling, bottom pipes rise from the ground), a small white ghost, and a dark slate footer bar showing "Score: N | High: N". The game uses the existing assets `assets/ghosty.png`, `assets/jump.wav`, and `assets/game_over.wav`. The Game also gives audio and visual feedback: sound effects for flapping, scoring, non-final hits, and game over; looping Background_Music; Screen_Shake on collisions; a Particle_Trail behind Ghosty; and visual score indicators. The scoring sound, the non-final hit sound, and the Background_Music have no asset files and are generated procedurally at runtime with the Web Audio API, so the Game needs no new asset files. All feedback is purely presentational and does not change the Physics_State, the Score, the Lives, or the gap sequence.

## Glossary

- **Game**: The Flappy Kiro application running in a web browser.
- **Canvas**: The rectangular play area in which the Game renders the sky, pipes, and Ghost.
- **Ghost**: The player-controlled character, rendered using `assets/ghosty.png`.
- **Pipe_Pair**: A top pipe and a bottom pipe separated by a vertical Gap, each with a darker cap at the Gap-facing end. Also referred to as a "wall pair" or "wall"; the terms are synonyms and this document uses Pipe_Pair.
- **Gap**: The vertical opening between the top pipe and the bottom pipe of a Pipe_Pair through which the Ghost passes.
- **Gap_Height**: The constant vertical size of the Gap, defined in the Difficulty_Config.
- **Gap_Center**: The vertical position at the middle of a Gap, measured from the top edge of the Canvas.
- **Max_Gap_Offset**: The maximum allowed absolute vertical difference between the Gap_Centers of two consecutively spawned Pipe_Pairs, defined in the Difficulty_Config.
- **Min_Pipe_Height**: The minimum visible height of each of the top pipe and the bottom pipe of a Pipe_Pair, which is 40 pixels.
- **Min_Clearance**: The minimum vertical margin, defined in the Difficulty_Config, by which the Gap_Height must exceed the Ghost's collision height.
- **Pipe_Spacing**: The horizontal distance in pixels between the leading (left) edges of two consecutively spawned Pipe_Pairs, defined in the Difficulty_Config.
- **First_Spawn_Delay**: The time in seconds between the Game_State changing to "Playing" and the spawn of the first Pipe_Pair, defined in the Difficulty_Config.
- **Pipe_Speed**: The current leftward speed of every Pipe_Pair in pixels per second. The Pipe_Speed is a single value shared by all Pipe_Pairs.
- **Initial_Pipe_Speed**: The Pipe_Speed at the start of a Round.
- **Speed_Increment**: The amount in pixels per second added to the Pipe_Speed each time the Score reaches a further multiple of the Speed_Interval.
- **Speed_Interval**: The number of points of Score between consecutive Pipe_Speed increases.
- **Max_Pipe_Speed**: The upper limit of the Pipe_Speed in pixels per second.
- **Difficulty_Config**: The single configuration location that holds every tunable obstacle constant (Gap_Height, Min_Clearance, Max_Gap_Offset, Pipe_Spacing, First_Spawn_Delay, Initial_Pipe_Speed, Speed_Increment, Speed_Interval, and Max_Pipe_Speed). Its values are separate from, and read together with, the Physics_Config.
- **Random_Source**: The component that supplies random numbers for Gap_Center selection. It can be replaced or seeded by a caller.
- **Previous_Pipe_Position**: The horizontal position of a Pipe_Pair before the most recent Physics_Step.
- **Cloud**: A semi-transparent white rounded decorative shape drawn in the sky background. A Cloud has a size and a horizontal speed, and has no effect on collisions or Score.
- **Score**: The number of Pipe_Pairs the Ghost has passed in the current round.
- **High_Score**: The highest Score achieved across rounds, persisted in the browser's local storage.
- **Score_Bar**: The footer bar below the Canvas that displays the text "Score: N | High: N".
- **Jump**: An upward velocity impulse applied to the Ghost.
- **Round**: One play session from start until a collision occurs.
- **Game_State**: One of "Ready", "Playing", or "Game_Over".
- **Jump_Input**: A press of the Space key, a mouse click on the Canvas, or a touch on the Canvas.
- **Audio_Player**: The Game component that plays `assets/jump.wav` and `assets/game_over.wav`.
- **Renderer**: The Game component that draws the Canvas and Score_Bar.
- **Storage**: The browser local storage used to persist the High_Score.
- **Physics_Config**: The single configuration location that holds every tunable physics constant (Gravity, Jump_Velocity, Max_Fall_Speed, Max_Rise_Speed, Physics_Step duration, and Max_Frame_Time).
- **Gravity**: A single named constant in the Physics_Config, expressed as a downward acceleration in pixels per second squared.
- **Jump_Velocity**: A single named constant in the Physics_Config, expressed as a fixed upward velocity in pixels per second.
- **Terminal_Velocity**: The maximum speed of the Ghost in each vertical direction. Max_Fall_Speed is the maximum downward speed and Max_Rise_Speed is the maximum upward speed, both in pixels per second.
- **Physics_Step**: One fixed-duration update of the Ghost's physics state, with the duration defined in the Physics_Config.
- **Physics_State**: The Ghost's vertical position and vertical velocity as computed by Physics_Steps. Collision detection and scoring use the Physics_State.
- **Previous_State**: The Physics_State as it was before the most recent Physics_Step.
- **Accumulator**: The elapsed time not yet consumed by Physics_Steps, which is always less than one Physics_Step duration after a frame update completes.
- **Interpolation**: The calculation of the Ghost's drawn position and tilt as a blend between the Previous_State and the current Physics_State, weighted by the Accumulator divided by the Physics_Step duration.
- **Max_Frame_Time**: The upper limit, defined in the Physics_Config, on the elapsed time that a single frame update adds to the Accumulator.
- **Collision_Config**: The single configuration location that holds every tunable collision, Lives, Invincibility, and animation constant (Ghost_Sprite_Width, Ghost_Sprite_Height, Hitbox_Width, Hitbox_Height, Hitbox_Offset_X, Hitbox_Offset_Y, Pipe_Body_Width, Pipe_Cap_Width, Pipe_Cap_Height, Initial_Lives, Invincibility_Duration, Blink_Interval, Knockback_Velocity, Hit_Flash_Duration, and Game_Over_Animation_Duration). Its values are separate from, and read together with, the Physics_Config and the Difficulty_Config.
- **Hitbox**: The axis-aligned rectangle that the Game uses for collision detection. The Ghost_Hitbox and each pipe rectangle are Hitboxes. All Hitboxes use the Canvas pixel coordinate system, with the origin at the top-left corner of the Canvas and the vertical axis pointing downward.
- **Ghost_Hitbox**: The Hitbox of the Ghost, a rectangle of Hitbox_Width by Hitbox_Height. The Ghost_Hitbox is a rectangular approximation of the rounded Ghost shape, inset inside the Ghost sprite, and its center is the Ghost's physics position plus Hitbox_Offset_X and Hitbox_Offset_Y.
- **Hitbox_Width, Hitbox_Height**: The width and height in pixels of the Ghost_Hitbox, defined in the Collision_Config. Hitbox_Height is the "Ghost's collision height" referred to in Requirement 14.
- **Hitbox_Offset_X, Hitbox_Offset_Y**: The horizontal and vertical offsets in pixels of the center of the Ghost_Hitbox from the center of the Ghost sprite, defined in the Collision_Config.
- **Ghost_Sprite_Width, Ghost_Sprite_Height**: The width and height in pixels of the Ghost sprite as drawn on the Canvas, defined in the Collision_Config.
- **Pipe_Rectangles**: The four Hitboxes of a Pipe_Pair: the top pipe body, the top pipe cap, the bottom pipe body, and the bottom pipe cap. Each cap is wider than the body of the same pipe.
- **Pipe_Body_Width, Pipe_Cap_Width, Pipe_Cap_Height**: The width of a pipe body, the width of a pipe cap, and the height of a pipe cap in pixels, defined in the Collision_Config. Pipe_Cap_Width is greater than Pipe_Body_Width, and the cap is horizontally centered on the body.
- **Playfield**: The region of the Canvas between the ceiling (the top edge of the Canvas, vertical coordinate 0) and the ground (the bottom edge of the Canvas).
- **Pipe_Hit**: An overlap between the Ghost_Hitbox and any Pipe_Rectangle that the Game counts as a collision, because the Ghost is not invincible.
- **Boundary_Hit**: A contact between the Ghost_Hitbox and the ground or the ceiling.
- **Lives**: The number of Pipe_Hits that the Ghost can still take in the current Round before the Game_State changes to "Game_Over". Lives is an integer from 0 to Initial_Lives.
- **Initial_Lives**: The number of Lives at the start of a Round, defined in the Collision_Config as an integer of at least 2. The initial default is chosen during design.
- **Invincibility**: The state, started by a non-final Pipe_Hit, in which the Game ignores overlaps between the Ghost_Hitbox and Pipe_Rectangles. Invincibility does not change the Game_State, which remains "Playing". The word "frames" in the context of Invincibility refers to this window.
- **Invincibility_Duration**: The length in seconds of Physics_Step time of the Invincibility window, defined in the Collision_Config.
- **Invincibility_Timer**: The remaining Invincibility time in seconds. The Ghost is invincible while the Invincibility_Timer is greater than 0.
- **Blink_Interval**: The duration in seconds for which the Ghost stays visible or hidden before the visibility alternates during Invincibility, defined in the Collision_Config.
- **Knockback**: A vertical velocity impulse of magnitude Knockback_Velocity applied to the Ghost on a non-final Pipe_Hit, directed away from the struck pipe.
- **Knockback_Velocity**: The magnitude in pixels per second of the Knockback, defined in the Collision_Config.
- **Hit_Animation**: The visual response to a non-final Pipe_Hit, consisting of the Ghost flash for Hit_Flash_Duration, the Knockback, and the Screen_Shake.
- **Hit_Flash_Duration**: The duration in seconds of the Ghost flash at the start of the Hit_Animation, defined in the Collision_Config.
- **Game_Over_Animation**: The visual response to the Game_State changing to "Game_Over", in which the Ghost spins and falls toward the ground, lasting Game_Over_Animation_Duration seconds of Physics_Step time.
- **Game_Over_Animation_Duration**: The duration in seconds of the Game_Over_Animation, defined in the Collision_Config.
- **Screen_Shake**: A cosmetic offset applied to the drawn Canvas world contents for Screen_Shake_Duration seconds after a Pipe_Hit or a Boundary_Hit. It is enabled by default through Screen_Shake_Enabled in the Feedback_Config, has an amplitude of at most Screen_Shake_Magnitude that decays over its duration, and is specified in Requirements 17 and 19.
- **Debug_Overlay**: An optional development view that draws the Ghost_Hitbox and the Pipe_Rectangles as outlines on the Canvas.
- **Swept_Test**: A collision test that detects an overlap occurring at any instant during one Physics_Step, assuming that the Ghost_Hitbox and each Pipe_Rectangle move linearly from their positions before the step to their positions after the step.
- **Feedback_Event**: A gameplay event that triggers feedback: a Jump, a point scored, a non-final Pipe_Hit, a final collision (the Game_State changing to "Game_Over"), a Pipe_Speed increase, and a new High_Score. Feedback_Events originating in the physics are produced by Physics_Steps and not by rendered frames.
- **Feedback_Config**: The single configuration location that holds every tunable feedback constant: Effects_Volume, Music_Volume, Music_Fade_Out_Duration, the source of each sound, Screen_Shake_Enabled, Screen_Shake_Duration, Screen_Shake_Magnitude, the Particle parameters, the Score_Popup parameters, Score_Pulse_Duration, Speed_Flash_Duration, and the Reduced_Motion settings. Its values are separate from, and read together with, the Physics_Config, the Difficulty_Config, and the Collision_Config.
- **Sound_Effect**: A short sound played in response to one Feedback_Event: the flap sound (`assets/jump.wav`), the score sound, the hit sound, or the game-over sound (`assets/game_over.wav`).
- **Procedural_Sound**: A sound that the Audio_Player synthesizes at runtime with the Web Audio API instead of loading a file. The score sound, the hit sound, and the Background_Music are Procedural_Sounds.
- **Background_Music**: A simple looping chiptune-style melody generated as a Procedural_Sound.
- **Effects_Volume**: The playback volume of every Sound_Effect, from 0 to 1, defined in the Feedback_Config.
- **Music_Volume**: The playback volume of the Background_Music, from 0 to 1, defined in the Feedback_Config and lower than the Effects_Volume.
- **Audio_Context**: The Web Audio API context that the Audio_Player uses to generate and play sound. Browsers keep it suspended until a user gesture occurs.
- **Muted**: The state of the Audio_Player, toggled by the M key, in which no Sound_Effect and no Background_Music is audible.
- **Shake_Offset**: The horizontal and vertical offset in Canvas pixels applied to the drawn world contents during a Screen_Shake.
- **Screen_Shake_Duration, Screen_Shake_Magnitude**: The length in seconds and the initial maximum amplitude in Canvas pixels of a Screen_Shake, defined in the Feedback_Config.
- **Particle**: A small, semi-transparent white circle that the Renderer draws as a decorative effect. A Particle has a position, a velocity, a size, an age, and a lifetime, and has no effect on collisions, Score, or physics.
- **Particle_Trail**: The stream of Particles emitted behind the Ghost while the Game_State is "Playing".
- **Effects_Random_Source**: A seedable random number source, separate from the Random_Source, that supplies every random number used for Particles. Drawing from the Effects_Random_Source never changes the Random_Source.
- **Score_Popup**: A floating "+1" text drawn near the Ghost when a point is scored. It rises and fades out over Score_Popup_Duration.
- **Score_Pulse**: A brief scale animation of the Score_Bar text lasting Score_Pulse_Duration after the Score changes.
- **Speed_Up_Flash**: A brief "Speed up!" text on the Canvas lasting Speed_Flash_Duration after the Pipe_Speed increases.
- **New_High_Indicator**: The highlight shown when the Score exceeds the High_Score that was stored when the Round started.
- **Reduced_Motion**: The browser preference "prefers-reduced-motion: reduce".

## Requirements

### Requirement 1: Browser Delivery and Ready Screen
**User Story:** As a player, I want to open the game in my browser, so that I can play without installing anything.

#### Acceptance Criteria

1. THE Game SHALL run in a web browser from static files (HTML, CSS, and JavaScript) without requiring a server-side component.
2. WHEN the Game page finishes loading, THE Game SHALL display the Canvas and the Score_Bar and set the Game_State to "Ready".
3. WHILE the Game_State is "Ready", THE Renderer SHALL display the Ghost at its starting position at the horizontal position of one third of the Canvas width and vertically centered, with no Pipe_Pairs moving.
4. WHILE the Game_State is "Ready", THE Renderer SHALL display a text prompt instructing the player to press Space, click, or tap to start.
5. IF `assets/ghosty.png` fails to load, THEN THE Renderer SHALL draw the Ghost as a white rounded shape of the same dimensions so that the Game remains playable.

### Requirement 2: Starting the Game
**User Story:** As a player, I want to start the game with a simple input, so that I can begin playing immediately.

#### Acceptance Criteria

1. WHEN a Jump_Input occurs WHILE the Game_State is "Ready", THE Game SHALL set the Game_State to "Playing" and apply a Jump to the Ghost.
2. WHEN the Game_State changes to "Playing", THE Game SHALL reset the Score to 0.
3. WHEN the Game_State changes to "Playing", THE Game SHALL set the Lives to Initial_Lives and the Invincibility_Timer to 0, as specified in Requirement 16.

### Requirement 3: Ghost Physics and Jump
**User Story:** As a player, I want the ghost to fall and jump with responsive physics, so that the game feels like a classic flap-style game.

#### Acceptance Criteria

1. WHILE the Game_State is "Playing", THE Game SHALL apply Gravity to the Ghost's vertical velocity on every Physics_Step, as specified in Requirement 13.
2. WHEN a Jump_Input occurs WHILE the Game_State is "Playing", THE Game SHALL apply a Jump to the Ghost as specified in Requirement 13, replacing any existing vertical velocity.
3. WHEN a Jump is applied, THE Audio_Player SHALL play the flap Sound_Effect (`assets/jump.wav`) as specified in Requirement 18.
4. WHILE the Game_State is "Playing", THE Game SHALL update the Ghost's vertical position on every Physics_Step, as specified in Requirement 13.
5. THE Game SHALL produce the same Ghost motion per unit of elapsed time on displays with refresh rates between 30 Hz and 144 Hz.
6. WHILE the Game_State is "Playing", THE Renderer SHALL tilt the Ghost upward when its vertical velocity is upward and downward when its vertical velocity is downward.

### Requirement 4: Pipe Generation
**User Story:** As a player, I want an endless stream of pipes to navigate, so that the game can continue as long as I survive.

#### Acceptance Criteria

1. WHILE the Game_State is "Playing", THE Game SHALL move all Pipe_Pairs leftward at the Pipe_Speed, as specified in Requirement 14.
2. WHILE the Game_State is "Playing", THE Game SHALL spawn each new Pipe_Pair at the right edge of the Canvas according to the Pipe_Spacing and First_Spawn_Delay, as specified in Requirement 14.
3. WHEN a Pipe_Pair spawns, THE Game SHALL choose the Gap_Center at random within the constraints specified in Requirement 14, so that the top pipe and the bottom pipe each have a visible height of at least the Min_Pipe_Height.
4. THE Game SHALL give every Pipe_Pair the same Gap_Height, as specified in Requirement 14, large enough for the Ghost to pass through when the Ghost is vertically aligned with the Gap_Center.
5. WHEN a Pipe_Pair has moved entirely beyond the left edge of the Canvas, THE Game SHALL remove the Pipe_Pair from the game.
6. THE Renderer SHALL draw each top pipe hanging from the top edge of the Canvas and each bottom pipe rising from the ground, each with a darker green cap at the Gap-facing end.

### Requirement 5: Scoring
**User Story:** As a player, I want to earn points for passing pipes, so that I can measure my progress.

#### Acceptance Criteria

1. WHEN the Ghost's horizontal position passes the right edge of a Pipe_Pair for the first time, THE Game SHALL increase the Score by 1.
2. THE Game SHALL increase the Score by at most 1 for each Pipe_Pair.
3. WHILE the Game_State is "Playing", THE Score_Bar SHALL display the current Score and the High_Score in the format "Score: N | High: N". The Score_Bar text format is unchanged by the Lives system; the Lives are displayed on the Canvas as specified in Requirement 16.
4. WHEN the Ghost's horizontal position passes the right edge of a Pipe_Pair, THE Game SHALL increase the Score by 1 for that Pipe_Pair regardless of whether the Invincibility_Timer is greater than 0, so that a Pipe_Pair passed during Invincibility scores exactly once. The right edge of a Pipe_Pair is the right edge of the wider of its Pipe_Rectangles.
5. WHILE the Game_State is "Game_Over", THE Game SHALL keep the Score unchanged.

### Requirement 6: Collision and Game Over
**User Story:** As a player, I want collisions to end the round, so that the game has challenge.

#### Acceptance Criteria

1. IF the Ghost_Hitbox overlaps any Pipe_Rectangle WHILE the Invincibility_Timer equals 0, THEN THE Game SHALL treat the overlap as a Pipe_Hit and reduce the Lives by 1 as specified in Requirement 16, and SHALL set the Game_State to "Game_Over" only WHEN the Lives reach 0.
2. IF the Ghost_Hitbox bottom edge reaches the ground (the bottom edge of the Canvas), THEN THE Game SHALL set the Game_State to "Game_Over" regardless of the Lives and the Invincibility_Timer, as specified in Requirement 15.
3. IF the Ghost_Hitbox top edge reaches the top edge of the Canvas, THEN THE Game SHALL set the Game_State to "Game_Over" regardless of the Lives and the Invincibility_Timer, as specified in Requirement 15.
4. WHEN the Game_State changes to "Game_Over", THE Audio_Player SHALL play the game-over Sound_Effect (`assets/game_over.wav`) once, as specified in Requirement 18. A non-final Pipe_Hit plays the hit Sound_Effect and not `assets/game_over.wav`.
5. WHEN the Game_State changes to "Game_Over", THE Game SHALL stop moving the Pipe_Pairs and the spawning of Pipe_Pairs, and SHALL apply Gravity to the Ghost only to run the Game_Over_Animation as specified in Requirement 17.
6. THE Game SHALL use a Ghost_Hitbox that lies strictly inside the Ghost's rendered sprite, as specified in Requirement 15.

### Requirement 7: High Score Persistence
**User Story:** As a player, I want my best score remembered, so that I can try to beat it.

#### Acceptance Criteria

1. WHEN the Game page finishes loading, THE Game SHALL read the High_Score from Storage and use 0 if no valid value is stored.
2. WHEN the Score exceeds the High_Score, THE Game SHALL set the High_Score equal to the Score.
3. WHEN the Game_State changes to "Game_Over", THE Game SHALL write the High_Score to Storage.
4. IF Storage is unavailable or a write fails, THEN THE Game SHALL keep the High_Score in memory for the current page session and continue operating.
5. FOR ALL High_Score values, writing the value to Storage and then reading it back SHALL produce the same value (round-trip property).

### Requirement 8: Restarting After Game Over
**User Story:** As a player, I want to restart quickly after losing, so that I can try again without reloading the page.

#### Acceptance Criteria

1. WHILE the Game_State is "Game_Over" and the Game_Over_Animation has completed, THE Renderer SHALL display a "Game Over" message, the final Score, and a prompt to restart.
2. WHEN a Jump_Input occurs WHILE the Game_State is "Game_Over" and at least 500 milliseconds have elapsed since the Game_State changed to "Game_Over", THE Game SHALL remove all Pipe_Pairs, reset the Ghost to its starting position and zero velocity, reset the Score to 0, reset the Lives to Initial_Lives, reset the Invincibility_Timer to 0, cancel any running Hit_Animation, Screen_Shake, and Game_Over_Animation, clear the Particles, Score_Popups, Score_Pulse, Speed_Up_Flash, and New_High_Indicator as specified in Requirement 19, and set the Game_State to "Ready".
3. WHEN a Jump_Input occurs WHILE the Game_State is "Game_Over" and fewer than 500 milliseconds have elapsed since the Game_State changed to "Game_Over", THE Game SHALL ignore the Jump_Input.

### Requirement 9: Retro Visual Style and Scaling
**User Story:** As a player, I want the game to look like the provided retro design, so that it has a consistent charming style.

#### Acceptance Criteria

1. THE Renderer SHALL draw a light-blue sky background with semi-transparent white rounded Clouds, matching the style shown in `img/example-ui.png`.
2. THE Renderer SHALL draw pipes in green with a darker green cap, matching the style shown in `img/example-ui.png`.
3. THE Renderer SHALL draw the Score_Bar below the Canvas with a dark slate background and bold white centered text.
4. THE Canvas SHALL have a fixed aspect ratio of approximately 4:3 (width to height).
5. WHILE the browser window is smaller than the Canvas, THE Game SHALL scale the Canvas and Score_Bar down proportionally to fit within the window without changing the aspect ratio.
6. THE Renderer SHALL render pixel positions and sizes consistently with the game logic so that visual overlap matches collision detection, and THE Game SHALL define every Hitbox in Canvas pixel coordinates so that Hitboxes are independent of the display scaling in criterion 5.

### Requirement 10: Sound Effects and Mute
**User Story:** As a player, I want sound effects that I can control, so that audio does not get in my way.

#### Acceptance Criteria

1. WHEN a Sound_Effect is requested, THE Audio_Player SHALL play the Sound_Effect from its start at the Effects_Volume, restarting it if it is already playing.
2. IF the browser blocks audio playback, the Web Audio API is unavailable, or an audio file fails to load, THEN THE Audio_Player SHALL skip the affected Sound_Effect or Background_Music and THE Game SHALL continue operating without error.
3. WHEN the player presses the M key, THE Audio_Player SHALL toggle between Muted and unmuted states, and the toggle SHALL apply to every Sound_Effect and to the Background_Music. The Game has a single mute toggle and no separate music mute key.
4. WHILE the Audio_Player is Muted, THE Audio_Player SHALL play no Sound_Effect and no Background_Music.
5. THE Audio_Player SHALL play the flap Sound_Effect and the game-over Sound_Effect from `assets/jump.wav` and `assets/game_over.wav`, and SHALL generate the score Sound_Effect, the hit Sound_Effect, and the Background_Music as Procedural_Sounds, so that no new asset file is required.
6. THE Audio_Player SHALL play the Sound_Effects and the Background_Music as specified in Requirement 18.

### Requirement 11: Pause and Resume
**User Story:** As a player, I want to be able to pause the game, so that I can take a break without losing progress.

#### Acceptance Criteria

1. WHEN the player presses the P key WHILE the Game_State is "Playing", THE Game SHALL pause all movement, physics, and spawning.
2. WHEN the player presses the P key WHILE the Game is paused, THE Game SHALL resume from the exact paused state.
3. WHILE the Game is paused, THE Renderer SHALL display a "Paused" message on the Canvas.
4. WHILE the Game is paused, THE Game SHALL ignore Jump_Input.
5. WHILE the Game is paused, THE Game SHALL hold the Invincibility_Timer, the Ghost blink state, the Hit_Animation, the Screen_Shake, and the Lives unchanged, and WHEN the Game resumes THE Game SHALL continue each of them from the value held at the pause.
6. WHILE the Game is paused, THE Game SHALL perform no collision detection.
7. WHILE the Game is paused, THE Game SHALL hold the Screen_Shake, the Particles, the Score_Popups, the Score_Pulse, and the Speed_Up_Flash unchanged and SHALL pause the Background_Music, as specified in Requirements 18 and 19.

### Requirement 12: Parallax Clouds
**User Story:** As a player, I want semi-transparent clouds that drift at different speeds, so that the sky looks deep and alive.

#### Acceptance Criteria

1. THE Renderer SHALL draw each Cloud as a white rounded shape with an opacity below 100 percent, so that the sky background and any distant objects behind the Cloud remain visible through the Cloud.
2. THE Game SHALL assign Clouds at least 3 distinct horizontal speeds, each lower than the horizontal speed of the Pipe_Pairs.
3. THE Renderer SHALL draw a Cloud with a lower horizontal speed at a smaller size than a Cloud with a higher horizontal speed, so that slower Clouds appear farther away and the Clouds create a parallax effect.
4. WHILE the Game_State is "Ready" or "Playing" and the Game is not paused, THE Game SHALL move every Cloud leftward at the horizontal speed assigned to that Cloud.
5. WHILE the Game_State is "Game_Over" or the Game is paused, THE Game SHALL hold every Cloud at its current position.
6. WHEN a Cloud has moved entirely beyond the left edge of the Canvas, THE Game SHALL reposition that Cloud just beyond the right edge of the Canvas so that the sky always contains Clouds.
7. THE Renderer SHALL draw Clouds behind the Pipe_Pairs and behind the Ghost.
8. WHEN a Game_State change from "Game_Over" to "Ready" occurs, THE Game SHALL keep every Cloud at its current position and assigned speed.

### Requirement 13: Physics System
**User Story:** As a player, I want the ghost to move with consistent, smooth, and predictable physics, so that the game feels fair and responsive on any display.

#### Acceptance Criteria

1. THE Physics_Config SHALL define Gravity, Jump_Velocity, Max_Fall_Speed, Max_Rise_Speed, the Physics_Step duration, and Max_Frame_Time as named constants in one location, and THE Game SHALL read every physics parameter from the Physics_Config only.
2. THE Physics_Config SHALL hold these initial tunable defaults, which a developer can change without modifying any other Game logic: Physics_Step duration 1/60 second (about 16.7 milliseconds), Max_Frame_Time 0.25 second, and values for Gravity, Jump_Velocity, Max_Fall_Speed, and Max_Rise_Speed chosen during design and playtesting.
3. WHILE the Game_State is "Playing", THE Game SHALL accelerate the Ghost downward by Gravity multiplied by the Physics_Step duration on every Physics_Step, so that the acceleration is defined per unit of time and not per frame.
4. WHEN a Jump_Input occurs WHILE the Game_State is "Playing" and the Game is not paused, THE Game SHALL set the Ghost's vertical velocity to Jump_Velocity upward, regardless of the previous vertical velocity.
5. THE Game SHALL treat a press of the Space key, a mouse click on the Canvas, and a touch on the Canvas as equivalent Jump_Inputs that each apply the same Jump_Velocity.
6. WHEN a held Space key generates repeated key events, THE Game SHALL apply a Jump only for the initial key press.
7. THE Game SHALL limit the Ghost's downward velocity to no more than Max_Fall_Speed and the Ghost's upward velocity to no more than Max_Rise_Speed after every Physics_Step, after every Jump, and after every Knockback.
8. WHEN a Physics_Step runs with no Jump_Input since the previous Physics_Step and the clamping limits are not reached, THE Game SHALL set the Ghost's new vertical velocity equal to the previous vertical velocity plus Gravity multiplied by the Physics_Step duration.
9. WHEN a Physics_Step runs, THE Game SHALL advance the Ghost's vertical position by the average of the previous and new vertical velocities multiplied by the Physics_Step duration.
10. THE Game SHALL change the Ghost's vertical velocity only through Gravity, a Jump, velocity clamping, a collision response (the Knockback of Requirement 17 or the Game_Over_Animation start), or a reset, and no other input or event SHALL alter the vertical velocity (velocity continuity).
11. THE Game SHALL advance the Physics_State only in whole Physics_Steps of fixed duration, and SHALL add the elapsed time of each frame to the Accumulator and run one Physics_Step for each full Physics_Step duration held in the Accumulator.
12. IF the elapsed time since the previous frame exceeds Max_Frame_Time, THEN THE Game SHALL add only Max_Frame_Time to the Accumulator, so that the Ghost does not pass through a pipe or the ground after a long pause such as a background browser tab.
13. WHEN a frame is rendered, THE Renderer SHALL draw the Ghost at a vertical position and tilt Interpolated between the Previous_State and the current Physics_State using the Accumulator divided by the Physics_Step duration as the blend weight.
14. THE Game SHALL perform collision detection (Requirements 6 and 15) and scoring (Requirement 5) using the Physics_State and the Previous_State and not the Interpolated drawn position.
15. WHILE the Game is paused, THE Game SHALL not add elapsed time to the Accumulator and SHALL keep the Ghost's Physics_State unchanged. WHILE the Game_State is "Game_Over" and the Game is not paused, THE Game SHALL continue to add elapsed time to the Accumulator and run Physics_Steps that advance only the Ghost's Physics_State for the Game_Over_Animation (Requirement 17), the animation timers, and the Clouds' held positions as specified in Requirement 12.
16. WHEN the Game resumes from a pause, THE Game SHALL discard the time elapsed during the pause so that the Accumulator does not receive that time.
17. WHEN the Game_State changes from "Game_Over" to "Ready", THE Game SHALL set the Ghost's vertical velocity to zero, set the Ghost's Physics_State and Previous_State to the starting position, and set the Accumulator to zero.
18. WHILE the Game_State is "Ready", THE Game SHALL hold the Ghost at its starting position with zero vertical velocity and apply no Gravity.
19. FOR ALL sequences of Jump_Inputs and frame elapsed times, running the physics twice from the same initial Physics_State with the same sequence SHALL produce identical Physics_States (determinism property).
20. FOR ALL splits of the same total elapsed time into different frame durations (each within Max_Frame_Time), THE Game SHALL produce the same Physics_State after the same number of whole Physics_Steps, independent of the display refresh rate.

### Requirement 14: Obstacle Generation and Progressive Difficulty
**User Story:** As a player, I want walls that are spaced, sized, and positioned fairly and that speed up as I score, so that every round is passable but gets steadily harder.

#### Acceptance Criteria

**Configuration**

1. THE Difficulty_Config SHALL define Gap_Height, Min_Clearance, Max_Gap_Offset, Pipe_Spacing, First_Spawn_Delay, Initial_Pipe_Speed, Speed_Increment, Speed_Interval, and Max_Pipe_Speed as named constants in one location, and THE Game SHALL read every obstacle parameter from the Difficulty_Config only.
2. THE Difficulty_Config SHALL hold initial tunable defaults for each constant, chosen during design and playtesting, which a developer can change without modifying any other Game logic.
3. THE Difficulty_Config SHALL require Initial_Pipe_Speed to be greater than 0, Max_Pipe_Speed to be greater than or equal to Initial_Pipe_Speed, Speed_Increment to be greater than or equal to 0, and Speed_Interval to be an integer greater than or equal to 1.

**Pipe spacing and first spawn**

4. WHEN the Game_State changes to "Playing", THE Game SHALL spawn the first Pipe_Pair at the right edge of the Canvas after First_Spawn_Delay seconds of Physics_Step time have elapsed.
5. WHILE the Game_State is "Playing", THE Game SHALL spawn each subsequent Pipe_Pair at the right edge of the Canvas when the leading (left) edge of the most recently spawned Pipe_Pair has moved left of its spawn position by Pipe_Spacing pixels.
6. THE Game SHALL keep Pipe_Spacing constant in pixels for the whole Round, so that the horizontal distance between consecutive Pipe_Pairs does not change when the Pipe_Speed increases.

**Gap size and passability**

7. THE Difficulty_Config SHALL require Gap_Height to be greater than or equal to the Ghost's collision height plus Min_Clearance.
8. THE Game SHALL give every Pipe_Pair the same Gap_Height.
9. THE Difficulty_Config SHALL require Max_Gap_Offset to be no greater than the vertical distance that the Ghost can travel, in either direction, under Gravity, Jump_Velocity, Max_Fall_Speed, and Max_Rise_Speed within the time that the Ghost needs to cover Pipe_Spacing at Max_Pipe_Speed (Pipe_Spacing divided by Max_Pipe_Speed).
10. IF the Difficulty_Config values violate any requirement of criteria 3, 7, or 9, or leave no valid Gap_Center range as defined in criterion 12, THEN THE Game SHALL report a descriptive configuration error and SHALL NOT start a Round.

**Random Gap positioning**

11. THE Game SHALL obtain every random number used for Gap_Center selection from the Random_Source, and THE Random_Source SHALL accept a caller-supplied seed or a caller-supplied replacement generator.
12. WHEN a Pipe_Pair spawns, THE Game SHALL choose the Gap_Center uniformly at random from the range whose lower bound is Min_Pipe_Height plus half of Gap_Height and whose upper bound is the Canvas height minus Min_Pipe_Height minus half of Gap_Height, so that the Gap lies fully between the top edge of the Canvas and the ground and each pipe is at least Min_Pipe_Height tall.
13. WHEN a Pipe_Pair other than the first spawns, THE Game SHALL further restrict the range of criterion 12 to Gap_Centers within Max_Gap_Offset of the Gap_Center of the previously spawned Pipe_Pair, and SHALL choose the Gap_Center uniformly at random from the restricted range.
14. WHEN the Random_Source is initialized with the same seed, THE Game SHALL generate the same sequence of Gap_Centers.

**Pipe speed and progressive increase**

15. WHEN the Game_State changes to "Playing", THE Game SHALL set the Pipe_Speed to Initial_Pipe_Speed.
16. WHILE the Game_State is "Playing" and the Game is not paused, THE Game SHALL move every Pipe_Pair leftward by the Pipe_Speed multiplied by the Physics_Step duration on every Physics_Step, so that the movement is defined per unit of time and not per frame.
17. THE Game SHALL apply the same Pipe_Speed to every Pipe_Pair on the Canvas, including Pipe_Pairs that were spawned before a Pipe_Speed increase.
18. WHEN the Score changes, THE Game SHALL set the Pipe_Speed to the lesser of Max_Pipe_Speed and Initial_Pipe_Speed plus Speed_Increment multiplied by the integer part of the Score divided by Speed_Interval.
19. THE Game SHALL keep the Pipe_Speed greater than or equal to its value at any earlier time in the same Round, and less than or equal to Max_Pipe_Speed.
20. WHEN the Game_State changes from "Game_Over" to "Ready", THE Game SHALL set the Pipe_Speed to Initial_Pipe_Speed.
21. WHILE the Game is paused, THE Game SHALL hold the Pipe_Speed unchanged, and WHEN the Game resumes THE Game SHALL continue with the Pipe_Speed held at the pause.
22. THE Game SHALL keep every Cloud speed lower than Initial_Pipe_Speed, so that the Cloud speeds remain lower than the Pipe_Speed for the whole Round as required by Requirement 12.

**Lifecycle, timing, and rendering**

23. WHEN a Pipe_Pair has moved entirely beyond the left edge of the Canvas, THE Game SHALL remove the Pipe_Pair, as specified in Requirement 4.
24. THE Game SHALL update Pipe_Pair positions, spawn timing, and the Pipe_Speed only in whole Physics_Steps using the Accumulator specified in Requirement 13, so that obstacle generation and movement are independent of the display refresh rate.
25. WHEN a frame is rendered, THE Renderer SHALL draw each Pipe_Pair at a horizontal position Interpolated between its Previous_Pipe_Position and its current position, using the same blend weight as the Ghost.
26. THE Game SHALL perform collision detection (Requirements 6 and 15) and scoring (Requirement 5) using the Pipe_Pair positions from the current Physics_State and the Previous_Pipe_Position, and not the Interpolated drawn positions.
27. WHEN the Game_State changes from "Game_Over" to "Ready", THE Game SHALL remove all Pipe_Pairs and reset the First_Spawn_Delay timing.

**Correctness properties**

28. FOR ALL Pipe_Pairs generated with any seed and any valid Difficulty_Config, the top pipe and the bottom pipe SHALL each have a visible height of at least Min_Pipe_Height and the Gap SHALL lie fully within the Canvas above the ground (invariant property).
29. FOR ALL consecutive Pipe_Pairs generated with any seed and any valid Difficulty_Config, the absolute difference between their Gap_Centers SHALL be less than or equal to Max_Gap_Offset (invariant property).
30. FOR ALL sequences of Score increases within a Round, the Pipe_Speed SHALL be monotonically non-decreasing and less than or equal to Max_Pipe_Speed (invariant property).
31. FOR ALL seeds, generating a sequence of Pipe_Pairs twice with the same seed and the same Difficulty_Config SHALL produce identical sequences (determinism property).
32. FOR ALL splits of the same total elapsed time into different frame durations (each within Max_Frame_Time), THE Game SHALL produce the same Pipe_Pair positions and Pipe_Speed after the same number of whole Physics_Steps (refresh-rate independence property).

### Requirement 15: Collision Detection
**User Story:** As a player, I want collisions to be precise and fair, so that the Ghost is hit only when it visibly touches a pipe, the ground, or the ceiling.

#### Acceptance Criteria

**Configuration and validation**

1. THE Collision_Config SHALL define Ghost_Sprite_Width, Ghost_Sprite_Height, Hitbox_Width, Hitbox_Height, Hitbox_Offset_X, Hitbox_Offset_Y, Pipe_Body_Width, Pipe_Cap_Width, and Pipe_Cap_Height as named constants in one location, and THE Game SHALL read every collision dimension from the Collision_Config only.
2. THE Collision_Config SHALL require Hitbox_Width to be less than Ghost_Sprite_Width, Hitbox_Height to be less than Ghost_Sprite_Height, and the Ghost_Hitbox, after applying Hitbox_Offset_X and Hitbox_Offset_Y, to lie entirely inside the Ghost sprite bounds with a margin greater than 0 on every side, so that the Ghost_Hitbox is strictly smaller than the sprite (extending Requirement 6, criterion 6).
3. THE Collision_Config SHALL require Pipe_Cap_Width to be greater than Pipe_Body_Width and Pipe_Cap_Height to be greater than 0.
4. IF the Collision_Config values violate criterion 2 or 3, THEN THE Game SHALL report a descriptive configuration error and SHALL NOT start a Round.

**Hitbox definitions**

5. THE Game SHALL define the Ghost_Hitbox as an axis-aligned rectangle, which approximates the rounded Ghost shape, centered on the Ghost's physics position (the vertical position from the Physics_State and the fixed horizontal position) plus Hitbox_Offset_X and Hitbox_Offset_Y.
6. THE Game SHALL keep the Ghost_Hitbox axis-aligned and unrotated, so that the visual tilt of the Ghost (Requirement 3, criterion 6), the Hit_Animation, and the Game_Over_Animation spin do not change the Ghost_Hitbox.
7. THE Game SHALL define the Pipe_Rectangles of every Pipe_Pair as four axis-aligned rectangles: a top body and a bottom body of width Pipe_Body_Width, and a top cap and a bottom cap of width Pipe_Cap_Width and height Pipe_Cap_Height, where each cap is horizontally centered on its body and placed at the Gap-facing end of the body.
8. THE Game SHALL position the Pipe_Rectangles from the Pipe_Pair's physics position and the Gap_Center, so that the top body and the top cap extend from the top edge of the Canvas to the top of the Gap, and the bottom body and the bottom cap extend from the bottom of the Gap to the ground.
9. THE Game SHALL compute the Ghost_Hitbox and the Pipe_Rectangles from the Physics_State only, independent of the Interpolation and independent of the display scaling.
10. THE Renderer SHALL draw the Ghost sprite and the pipes at sizes equal to the Ghost_Sprite_Width, the Ghost_Sprite_Height, the Pipe_Body_Width, the Pipe_Cap_Width, and the Pipe_Cap_Height, so that the drawn pipe shapes match the Pipe_Rectangles.

**Wall (pipe) collision boundaries**

11. THE Game SHALL define an overlap between the Ghost_Hitbox and a Pipe_Rectangle as an intersection with an area greater than 0, so that rectangles that only touch along an edge or at a corner do not overlap. This strict definition favors the player, because a touch of exact edges cannot be distinguished visually and floating-point positions would otherwise make the result unpredictable.
12. WHEN a Physics_Step runs WHILE the Game_State is "Playing", THE Game SHALL test the Ghost_Hitbox against all four Pipe_Rectangles of every Pipe_Pair, so that the top body, the top cap, the bottom body, and the bottom cap are each tested.
13. WHEN a Physics_Step runs, THE Game SHALL apply the Swept_Test to the Ghost_Hitbox and each Pipe_Rectangle, using the positions before and after the step, so that the Ghost cannot pass through a Pipe_Rectangle between two Physics_Steps at any Pipe_Speed up to Max_Pipe_Speed and any vertical speed up to Max_Fall_Speed or Max_Rise_Speed.
14. IF the Swept_Test finds an overlap at any instant of a Physics_Step, THEN THE Game SHALL report the overlap for that Physics_Step, regardless of whether the Ghost_Hitbox and the Pipe_Rectangle overlap at the end of the step.

**Ground and ceiling detection**

15. THE Game SHALL treat the ground as the bottom edge of the Playfield (the bottom edge of the Canvas) and the ceiling as the top edge of the Playfield (vertical coordinate 0), and SHALL test the Ghost_Hitbox and not the Ghost sprite against both.
16. IF the bottom edge of the Ghost_Hitbox is at or below the ground, or the top edge of the Ghost_Hitbox is at or above the ceiling, THEN THE Game SHALL register a Boundary_Hit. The inclusive comparison differs from the strict pipe overlap because the Playfield edges are limits of movement and not surfaces that the Ghost can graze.
17. IF a Boundary_Hit is registered, THEN THE Game SHALL set the Game_State to "Game_Over" regardless of the Lives and the Invincibility_Timer.
18. WHEN a Boundary_Hit is registered, THE Game SHALL clamp the Ghost's vertical position so that the Ghost_Hitbox lies within the Playfield, and THE Renderer SHALL draw the Ghost sprite so that the sprite does not extend beyond the ceiling or the ground.
19. IF a Boundary_Hit and a Pipe_Hit are both detected in the same Physics_Step, THEN THE Game SHALL treat the step as a Boundary_Hit only and SHALL NOT reduce the Lives.

**Debug overlay (optional)**

20. WHERE the Debug_Overlay is available, THE Game SHALL toggle the Debug_Overlay when the player presses the H key, with the Debug_Overlay off by default.
21. WHILE the Debug_Overlay is on, THE Renderer SHALL draw the Ghost_Hitbox and every Pipe_Rectangle as outlines at the positions used for collision detection, and the Debug_Overlay SHALL have no effect on collision results, Score, or Lives.

**Correctness properties**

22. FOR ALL valid Collision_Configs, the Ghost_Hitbox SHALL lie strictly inside the Ghost sprite bounds for every Ghost position (invariant property).
23. FOR ALL valid Difficulty_Configs, Physics_Configs, and Collision_Configs, and FOR ALL Ghost positions and velocities, a Ghost_Hitbox that overlaps a Pipe_Rectangle at any instant of a Physics_Step SHALL be reported by the Swept_Test for that step (no-tunneling property).
24. FOR ALL sequences of Jump_Inputs, frame elapsed times, and seeds, running the collision detection twice from the same initial state SHALL produce identical collision results (determinism property).
25. FOR ALL splits of the same total elapsed time into different frame durations (each within Max_Frame_Time), THE Game SHALL produce the same collision results after the same number of whole Physics_Steps (refresh-rate independence property).

### Requirement 16: Lives and Invincibility
**User Story:** As a player, I want several lives and a short period of protection after a hit, so that one mistake does not end the round and I have time to recover.

#### Acceptance Criteria

**Lives**

1. THE Collision_Config SHALL define Initial_Lives as an integer of at least 2, with an initial default chosen during design and playtesting, which a developer can change without modifying any other Game logic.
2. WHEN the Game_State changes to "Playing", THE Game SHALL set the Lives to Initial_Lives.
3. WHEN a Pipe_Hit occurs, THE Game SHALL reduce the Lives by exactly 1.
4. WHEN a Pipe_Hit occurs and the Lives after the reduction are greater than 0, THE Game SHALL set the Invincibility_Timer to Invincibility_Duration, apply the Knockback, and start the Hit_Animation as specified in Requirement 17.
5. WHEN a Pipe_Hit occurs and the Lives after the reduction equal 0, THE Game SHALL set the Game_State to "Game_Over".
6. WHILE the Game_State is "Game_Over", THE Game SHALL keep the Lives unchanged.
7. THE Game SHALL keep the Lives between 0 and Initial_Lives inclusive.
8. THE Renderer SHALL display the Lives as heart icons in a heads-up display in the top-left corner of the Canvas, with one filled heart for each remaining Life and one outlined heart for each lost Life, WHILE the Game_State is "Ready", "Playing", or "Game_Over". The Score_Bar text format "Score: N | High: N" SHALL remain unchanged, so that the Score_Bar does not display the Lives.

**Invincibility**

9. THE Collision_Config SHALL define Invincibility_Duration (greater than 0 seconds) and Blink_Interval (greater than 0 seconds and less than or equal to Invincibility_Duration) as named constants, with initial defaults chosen during design and playtesting.
10. WHILE the Invincibility_Timer is greater than 0, THE Game SHALL ignore every overlap between the Ghost_Hitbox and the Pipe_Rectangles, so that no Life is lost and no Hit_Animation is re-triggered.
11. WHILE the Invincibility_Timer is greater than 0, THE Game SHALL continue to detect a Boundary_Hit and set the Game_State to "Game_Over" as specified in Requirement 15.
12. WHILE the Invincibility_Timer is greater than 0, THE Game SHALL process Jump_Input in the same way as when the Invincibility_Timer equals 0, as specified in Requirement 13.
13. WHILE the Game_State is "Playing" and the Game is not paused, THE Game SHALL decrease the Invincibility_Timer by the Physics_Step duration on every Physics_Step, down to a minimum of 0, so that the Invincibility_Timer is measured in Physics_Step time and is independent of the display refresh rate.
14. THE Game SHALL end the Invincibility in the Physics_Step in which the Invincibility_Timer reaches 0, and SHALL treat an overlap detected in that Physics_Step and in later Physics_Steps as a Pipe_Hit.
15. WHILE the Invincibility_Timer is greater than 0, THE Game SHALL NOT change the Invincibility_Timer in response to any overlap, so that further overlaps do not extend the Invincibility.
16. WHILE the Invincibility_Timer is greater than 0, THE Renderer SHALL alternate the visibility of the Ghost sprite every Blink_Interval seconds, starting with the Ghost hidden, and WHEN the Invincibility_Timer reaches 0 THE Renderer SHALL draw the Ghost visible.
17. IF the Ghost_Hitbox still overlaps a Pipe_Rectangle in the Physics_Step in which the Invincibility ends, THEN THE Game SHALL treat the overlap as a new Pipe_Hit and reduce the Lives by 1. The Knockback is intended to push the Ghost clear of the pipe before the Invincibility ends, and the Game SHALL NOT extend the Invincibility to compensate for a Ghost that remains in a pipe.
18. WHEN the Game_State changes from "Game_Over" to "Ready", THE Game SHALL set the Invincibility_Timer to 0 and the Lives to Initial_Lives, as specified in Requirement 8.
19. WHILE the Game is paused, THE Game SHALL hold the Invincibility_Timer unchanged, as specified in Requirement 11.

**Correctness properties**

20. FOR ALL Pipe_Hits that occur while the Invincibility_Timer equals 0, THE Game SHALL reduce the Lives by exactly 1 (invariant property).
21. FOR ALL sequences of Physics_Steps, the Lives SHALL be greater than or equal to 0 and less than or equal to Initial_Lives (invariant property).
22. FOR ALL Physics_Steps in which the Invincibility_Timer is greater than 0 at the time of the overlap test, THE Game SHALL leave the Lives unchanged by Pipe_Rectangle overlaps (invariant property).
23. THE Game SHALL set the Game_State to "Game_Over" if and only if the Lives reach 0 or a Boundary_Hit is registered (invariant property).
24. FOR ALL sequences of Physics_Steps without a Pipe_Hit, the Invincibility_Timer SHALL be non-increasing and SHALL reach 0 after exactly the number of whole Physics_Steps needed to consume Invincibility_Duration (invariant property).
25. FOR ALL seeds and sequences of Jump_Inputs, running the Lives and Invincibility logic twice from the same initial state SHALL produce identical Lives and Invincibility_Timer values after every Physics_Step (determinism property).

### Requirement 17: Collision Response Animations
**User Story:** As a player, I want clear visual feedback when the Ghost is hit or the round ends, so that I understand what happened.

#### Acceptance Criteria

**Hit animation**

1. THE Collision_Config SHALL define Knockback_Velocity, Hit_Flash_Duration, and Game_Over_Animation_Duration as named constants, with initial defaults chosen during design and playtesting, and THE Collision_Config SHALL require Hit_Flash_Duration to be less than or equal to Invincibility_Duration and Knockback_Velocity to be less than or equal to the applicable Max_Rise_Speed or Max_Fall_Speed.
2. WHEN a non-final Pipe_Hit occurs, THE Game SHALL set the Ghost's vertical velocity to Knockback_Velocity directed upward if the center of the Ghost_Hitbox is below the Gap_Center of the struck Pipe_Pair, and directed downward if the center is at or above the Gap_Center, so that the Ghost moves away from the struck pipe, replacing any existing vertical velocity in the same way as a Jump (Requirement 13, criterion 4).
3. WHEN the Knockback is applied, THE Game SHALL clamp the Ghost's vertical velocity to Max_Fall_Speed and Max_Rise_Speed as specified in Requirement 13, criterion 7, and SHALL treat the Knockback as a collision response permitted by Requirement 13, criterion 10.
4. WHEN a non-final Pipe_Hit occurs, THE Renderer SHALL tint the Ghost white for Hit_Flash_Duration seconds, starting at the Physics_Step of the Pipe_Hit.
5. WHEN a non-final Pipe_Hit occurs or the Game_State changes to "Game_Over" because of a final Pipe_Hit or a Boundary_Hit, THE Game SHALL start a Screen_Shake as specified in Requirement 19, criteria 1 to 8. The Screen_Shake is enabled by default, its duration, magnitude, and enable toggle are defined in the Feedback_Config, and its offset is deterministic and does not use the Random_Source, so that the Screen_Shake does not change the Gap_Center sequence of Requirement 14, criterion 14.
6. THE Game SHALL measure the duration of the Hit_Animation, the Screen_Shake, and the Game_Over_Animation in Physics_Step time, so that the animations are frozen while the Game is paused (Requirement 11) and are independent of the display refresh rate.
7. THE Game SHALL compute every collision result from the Physics_State only, so that the Hit_Animation, the Screen_Shake, the blink, and the Game_Over_Animation do not change collision results.
8. WHEN a non-final Pipe_Hit occurs, THE Audio_Player SHALL play the hit Sound_Effect and no other Sound_Effect, as specified in Requirement 18. The Game SHALL use only the existing asset files `assets/ghosty.png`, `assets/jump.wav`, and `assets/game_over.wav`, because the hit Sound_Effect is a Procedural_Sound.

**Game-over animation**

9. WHEN the Game_State changes to "Game_Over", THE Game SHALL set the Ghost's vertical velocity to 0, and THE Game SHALL start the Game_Over_Animation, in which the Ghost spins and falls under Gravity, clamped to Max_Fall_Speed, until the Ghost_Hitbox rests on the ground or Game_Over_Animation_Duration seconds have elapsed.
10. WHILE the Game_Over_Animation runs, THE Game SHALL keep the Pipe_Pairs stationary, SHALL spawn no Pipe_Pairs, SHALL perform no Pipe_Hit detection, and SHALL keep the Score and the Lives unchanged.
11. WHILE the Game_Over_Animation runs, THE Game SHALL clamp the Ghost's vertical position so that the Ghost_Hitbox does not move below the ground.
12. WHEN the Game_Over_Animation completes, THE Renderer SHALL display the "Game Over" message as specified in Requirement 8, criterion 1.
13. WHEN the Game_State changes to "Game_Over", THE Game SHALL apply the 500 millisecond restart lockout of Requirement 8 measured from the moment the Game_State changed, independent of the progress of the Game_Over_Animation.
14. WHEN a Jump_Input restarts the Game under Requirement 8, criterion 2, WHILE the Game_Over_Animation is still running, THE Game SHALL cancel the Game_Over_Animation.
15. WHEN the Game_State changes from "Game_Over" to "Ready", THE Game SHALL reset the Ghost's rotation, the Ghost's visibility, the Hit_Animation, and the Screen_Shake to their initial states.

**Correctness properties**

16. FOR ALL Ghost velocities before a non-final Pipe_Hit, THE Ghost's vertical velocity after the Knockback SHALL have a magnitude less than or equal to the applicable Max_Fall_Speed or Max_Rise_Speed (invariant property).
17. FOR ALL runs with the same seed and the same sequence of Jump_Inputs and frame elapsed times, THE Game SHALL produce identical Ghost states, Lives, and animation timer values after the same number of whole Physics_Steps (determinism property).

### Requirement 18: Audio Feedback and Background Music
**User Story:** As a player, I want sound effects for my actions and background music, so that the game feels lively and I get immediate audio feedback.

#### Acceptance Criteria

**Sound effects**

1. WHEN a Jump is applied, including the starting Jump of Requirement 2, criterion 1, THE Audio_Player SHALL play the flap Sound_Effect from `assets/jump.wav` exactly once.
2. WHEN the Score increases by 1, THE Audio_Player SHALL play the score Sound_Effect exactly once.
3. WHEN a non-final Pipe_Hit occurs, THE Audio_Player SHALL play the hit Sound_Effect exactly once.
4. WHEN the Game_State changes to "Game_Over", THE Audio_Player SHALL play the game-over Sound_Effect from `assets/game_over.wav` exactly once, whether the cause is a final Pipe_Hit, a ground Boundary_Hit, or a ceiling Boundary_Hit.
5. WHEN the Game_State changes to "Game_Over" because of a final Pipe_Hit, THE Audio_Player SHALL play the game-over Sound_Effect and SHALL NOT play the hit Sound_Effect for the same Physics_Step.
6. THE Game SHALL request the score Sound_Effect, the hit Sound_Effect, and the game-over Sound_Effect from the Feedback_Events produced by Physics_Steps, and not from rendered frames, so that the number and the order of sound requests are independent of the display refresh rate.
7. WHEN several Feedback_Events occur within a short time, THE Audio_Player SHALL play one Sound_Effect for each Feedback_Event, restarting any Sound_Effect that is still playing, without raising an error.
8. THE Audio_Player SHALL play every Sound_Effect at the Effects_Volume.
9. THE Audio_Player SHALL read the source of each Sound_Effect (an asset file or a Procedural_Sound) from the Feedback_Config, so that a Procedural_Sound can later be replaced by an asset file without modifying other Game logic.
10. WHILE the Game is paused, THE Audio_Player SHALL play no Sound_Effect.
11. WHILE the Audio_Player is Muted, THE Audio_Player SHALL play no Sound_Effect.

**Audio start-up**

12. WHEN the first Jump_Input or key press occurs, THE Audio_Player SHALL create or resume the Audio_Context, so that audio starts only after a user gesture as browsers require.
13. IF the Audio_Context cannot be created or resumed, THEN THE Audio_Player SHALL continue silently, and THE Game SHALL continue operating without error.

**Background music**

14. WHEN the Game_State changes to "Playing", THE Audio_Player SHALL start the Background_Music from its beginning at the Music_Volume.
15. WHILE the Game_State is "Playing" and the Game is not paused, THE Audio_Player SHALL loop the Background_Music without an audible gap at the loop point.
16. WHEN the Game is paused, THE Audio_Player SHALL pause the Background_Music, and WHEN the Game resumes, THE Audio_Player SHALL resume the Background_Music from the point at which the pause occurred.
17. WHEN the Game_State changes to "Game_Over", THE Audio_Player SHALL fade the Background_Music out over Music_Fade_Out_Duration seconds and then stop the Background_Music.
18. WHEN the Game_State changes from "Game_Over" to "Ready", THE Audio_Player SHALL keep the Background_Music stopped until the Game_State changes to "Playing" again.
19. WHILE the Audio_Player is Muted, THE Audio_Player SHALL keep the Background_Music silent, and WHEN the player unmutes WHILE the Game_State is "Playing" and the Game is not paused, THE Audio_Player SHALL make the Background_Music audible again.
20. THE Feedback_Config SHALL define Music_Volume and Effects_Volume as named constants, with Music_Volume lower than Effects_Volume, and THE Game SHALL read the volumes from the Feedback_Config only.

**Correctness properties**

21. FOR ALL sequences of Feedback_Events while the Audio_Player is not Muted and the Game is not paused, the number of Sound_Effect play requests SHALL equal the number of Jumps plus the number of points scored plus the number of non-final Pipe_Hits plus the number of Game_State changes to "Game_Over" (counting property).
22. FOR ALL sequences of Feedback_Events while the Audio_Player is Muted or the Game is paused, the number of audible Sound_Effect play requests SHALL be 0 (invariant property).
23. FOR ALL runs with the same seed, the same sequence of Jump_Inputs, and the same frame elapsed times, THE Game SHALL produce the same sequence of Sound_Effect play requests (determinism property).

### Requirement 19: Visual Feedback Effects
**User Story:** As a player, I want screen shake, a particle trail, and visual score indicators, so that hits and points feel impactful and I can see my progress at a glance.

#### Acceptance Criteria

**Screen shake**

1. WHERE Screen_Shake_Enabled in the Feedback_Config is true, WHEN a non-final Pipe_Hit occurs or the Game_State changes to "Game_Over", THE Renderer SHALL apply a Screen_Shake for Screen_Shake_Duration seconds. Screen_Shake_Enabled is true by default.
2. THE Feedback_Config SHALL define Screen_Shake_Duration (greater than 0 seconds) and Screen_Shake_Magnitude (greater than 0 pixels) as named constants with initial tunable defaults chosen during design and playtesting.
3. WHILE a Screen_Shake runs, THE Renderer SHALL compute the Shake_Offset deterministically from the number of Screen_Shakes started in the Round and the elapsed shake time, and SHALL NOT use `Math.random`, the Random_Source, or the Effects_Random_Source.
4. WHILE a Screen_Shake runs, THE Renderer SHALL keep the magnitude of the Shake_Offset less than or equal to a maximum amplitude that decreases from Screen_Shake_Magnitude toward 0 over Screen_Shake_Duration, and WHEN the elapsed shake time reaches Screen_Shake_Duration THE Renderer SHALL apply a Shake_Offset of 0.
5. WHILE a Screen_Shake runs, THE Renderer SHALL apply the Shake_Offset to the world contents drawn on the Canvas (sky, Clouds, pipes, Ghost, Particles, and Score_Popups), and SHALL NOT apply the Shake_Offset to the Score_Bar, the heart icons of Requirement 16, criterion 8, or the text overlays ("Ready" prompt, "Paused", "Game Over", and Speed_Up_Flash).
6. IF a Screen_Shake starts while another Screen_Shake is running, THEN THE Game SHALL restart the Screen_Shake from its full magnitude and not add the two offsets.
7. THE Game SHALL measure the Screen_Shake in Physics_Step time, SHALL hold the Screen_Shake while the Game is paused, and SHALL cancel the Screen_Shake when the Game_State changes from "Game_Over" to "Ready".
8. THE Game SHALL leave the Ghost_Hitbox, the Pipe_Rectangles, the collision results, and the Physics_State unchanged by the Screen_Shake, and SHALL define the Shake_Offset in Canvas pixels so that the display scaling of Requirement 9, criterion 5 scales the offset proportionally.

**Reduced motion**

9. WHERE the browser reports Reduced_Motion, THE Game SHALL disable the Screen_Shake and the Score_Pulse and SHALL reduce the Particle emission to the fraction of the normal rate and the normal count cap defined as the Reduced_Motion particle factor in the Feedback_Config.

**Particle trail**

10. WHILE the Game_State is "Playing" and the Game is not paused, THE Game SHALL emit Particles behind the Ghost at the Particle_Emit_Rate, in Particles per second of Physics_Step time, defined in the Feedback_Config.
11. WHEN a Jump is applied WHILE the Game_State is "Playing", THE Game SHALL emit an additional burst of Particles whose count is defined as Particle_Flap_Burst_Count in the Feedback_Config.
12. WHILE the Ghost is hidden by the Invincibility blink of Requirement 16, criterion 16, THE Game SHALL emit no Particles.
13. THE Feedback_Config SHALL define Particle_Lifetime, Particle_Size, Particle_Drift, Particle_Max_Count, Particle_Emit_Rate, and Particle_Flap_Burst_Count as named constants with initial tunable defaults chosen during design and playtesting.
14. THE Game SHALL give every Particle a position that moves leftward at the Pipe_Speed multiplied by a scroll factor defined in the Feedback_Config, plus the Particle_Drift, and SHALL advance the position and age of every Particle only in whole Physics_Steps, so that the trail appears to stay behind the Ghost as the world scrolls.
15. THE Renderer SHALL draw every Particle as a semi-transparent white circle whose opacity decreases from its initial value to 0 over its Particle_Lifetime, and SHALL draw the Particles in front of the Clouds and the Pipe_Pairs and behind the Ghost.
16. WHEN a Particle reaches the end of its Particle_Lifetime, THE Game SHALL remove the Particle.
17. IF the number of Particles would exceed Particle_Max_Count, THEN THE Game SHALL remove the oldest Particles until the number equals Particle_Max_Count.
18. THE Game SHALL obtain every random number used for Particles from the Effects_Random_Source, which accepts a caller-supplied seed or a caller-supplied replacement generator, and SHALL NOT draw from the Random_Source for Particles, so that the Gap_Center sequence of Requirement 14, criterion 14 is the same whether the Particles are enabled or disabled.
19. WHILE the Game is paused, THE Game SHALL hold every Particle unchanged, and WHILE the Game_State is "Game_Over" THE Game SHALL emit no new Particles and let the existing Particles age and fade out.
20. WHEN the Game_State changes from "Game_Over" to "Ready", THE Game SHALL remove every Particle.
21. THE Game SHALL leave collision results, the Score, the Lives, and the Physics_State unchanged by the Particles.

**Score indicators**

22. WHEN the Score increases by 1, THE Game SHALL create a Score_Popup with the text "+1" at the Ghost's position, and THE Renderer SHALL raise the Score_Popup and fade its opacity from its initial value to 0 over Score_Popup_Duration, drawn in front of the Ghost.
23. WHEN a Score_Popup reaches the end of Score_Popup_Duration, THE Game SHALL remove the Score_Popup.
24. WHEN the Score changes, THE Renderer SHALL scale the Score_Bar text up briefly and back to its normal size over Score_Pulse_Duration, with the Score_Bar text format "Score: N | High: N" unchanged. The Score_Bar remains the main score display, and the Game draws no additional large score display on the Canvas.
25. WHEN the Score first exceeds the High_Score that was stored when the Round started, and that stored High_Score is greater than 0, THE Renderer SHALL show the New_High_Indicator by highlighting the "High: N" part of the Score_Bar in a distinct color for the rest of the Round.
26. WHEN the Game_State changes to "Game_Over" and the New_High_Indicator was shown in the Round, THE Renderer SHALL display the text "New High!" together with the "Game Over" message of Requirement 8, criterion 1.
27. WHEN the Pipe_Speed increases because the Score reaches a further multiple of Speed_Interval as specified in Requirement 14, criterion 18, THE Renderer SHALL display the Speed_Up_Flash with the text "Speed up!" for Speed_Flash_Duration seconds.
28. THE Game SHALL measure the Score_Popup, the Score_Pulse, and the Speed_Up_Flash in Physics_Step time, SHALL hold them while the Game is paused, and SHALL clear them when the Game_State changes from "Game_Over" to "Ready".
29. THE Game SHALL leave the Score, the Lives, the Pipe_Speed, and the Physics_State unchanged by the Score_Popups, the Score_Pulse, the Speed_Up_Flash, and the New_High_Indicator.

**Correctness properties**

30. FOR ALL sequences of Physics_Steps, the number of Particles SHALL be less than or equal to Particle_Max_Count (invariant property).
31. FOR ALL Particles, the opacity SHALL be non-increasing over the Particle's life, and the Particle SHALL be removed once its age reaches its Particle_Lifetime (invariant property).
32. FOR ALL Screen_Shakes, the magnitude of the Shake_Offset SHALL be non-increasing over the Screen_Shake and SHALL be 0 after Screen_Shake_Duration (invariant property).
33. FOR ALL seeds, THE Game SHALL produce the same Gap_Center sequence with the Particles, the Screen_Shake, and the other visual effects enabled as with them disabled (determinism property).

### Requirement 20: Feedback Configuration and Presentation-Only Guarantee
**User Story:** As a developer, I want all feedback settings in one place and guaranteed not to affect gameplay, so that I can tune the feel without risking game logic.

#### Acceptance Criteria

1. THE Feedback_Config SHALL define Effects_Volume, Music_Volume, Music_Fade_Out_Duration, the source of each Sound_Effect, Screen_Shake_Enabled, Screen_Shake_Duration, Screen_Shake_Magnitude, the Particle parameters of Requirement 19, criterion 13, the Reduced_Motion particle factor, Score_Popup_Duration, Score_Pulse_Duration, and Speed_Flash_Duration as named constants in one location, and THE Game SHALL read every feedback parameter from the Feedback_Config only.
2. THE Feedback_Config SHALL hold initial tunable defaults for each constant, chosen during design and playtesting, which a developer can change without modifying any other Game logic.
3. IF the Feedback_Config has a volume outside the range 0 to 1, a Music_Volume that is not lower than the Effects_Volume, a duration or magnitude that is not greater than 0, or a Particle_Max_Count below 1, THEN THE Game SHALL report a descriptive configuration error and SHALL NOT start a Round.
4. THE Game SHALL leave the Physics_State, the Score, the Lives, the collision results, and the Gap_Center sequence unchanged by any feedback effect (presentation-only property).
5. FOR ALL runs with the same seed, the same sequence of Jump_Inputs, and the same frame elapsed times, THE Game SHALL produce identical Physics_States, Scores, and Lives with all feedback effects enabled and with all feedback effects disabled (presentation-only property).
6. FOR ALL splits of the same total elapsed time into different frame durations (each within Max_Frame_Time), THE Game SHALL produce the same Particles, Score_Popups, and Screen_Shake state after the same number of whole Physics_Steps (refresh-rate independence property).
