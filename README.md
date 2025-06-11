import pygame
import random
import sys

# Display dimensions
display_width = 850
display_height = 610

# Initialize game display
gameDisplay = pygame.display.set_mode((display_width, display_height))
pygame.display.set_caption('Snake And Ladder')

# Clock for controlling game speed
clock = pygame.time.Clock()

# Game variables
first_roll = False
roll1 = roll2 = None
crashed = False
pause = True
size = 25
roll = False
DONE1 = False

# Colors
black = (0, 0, 0)
white = (255, 255, 255)
back = (96, 107, 114)
forg = (255, 138, 119)
darkback = (53, 53, 53)
boardclr = (255, 199, 95)
boardclr2 = (255, 237, 203)
snkclr = (168, 187, 92)
ladclr = (195, 64, 54)
pla1clr = (0, 211, 255)
pla2clr = (255, 121, 191)

MENU_OPTIONS = ["Start Game", "Continue", "Quit"]

# Load board image
board = pygame.image.load('board.jpeg')
board_rect = board.get_rect()
board_rect.topleft = (0, 0) 



ladders = ((1, 38), (4, 14), (9, 31), (21, 42), (28, 84), (51, 67), (71, 91), (80, 100))
snakes = ((98, 79), (95, 75), (93, 73), (87, 24), (64, 60), (62, 19), (54, 34), (17, 7))
syntax_error = [1, 6, 6, 1, 4, 6, 3]
#syntax_error = [4,3, 6, 1, 4, 6, 3]
syntax_count = 0

# Board mapping
board_map = []
n = 1
m = 100
for i in range(1, 11):
    if i % 2 != 0:
        for j in range(1, 11):
            board_map.append((((j * 60) - 30) + ((j * 60) - 30) // 60, ((n * 60) - 30) + ((n * 60) - 30) // 60, m))
            m -= 1
        n += 1
    else:
        for j in range(10, 0, -1):
            board_map.append((((j * 60) - 30) + ((j * 60) - 30) // 60, ((n * 60) - 30) + ((n * 60) - 30) // 60, m))
            m -= 1
        n += 1
board_map.append((700, 5700, 0))  # Including an out-of-bound marker (5700 seems a bit high, verify if this is intended)

def dice_sound():
    pygame.mixer.init()
    dice_sound=pygame.mixer.Sound('dice_sound.mp3')
    dice_sound.set_volume(1)
    dice_sound.play()

def Victory_Sound():
        pygame.mixer.init()                          # Initialize pygame audio mixer
        victory_sound = pygame.mixer.Sound("victory_sound.mp3")  # Load bomb sound effect
        victory_sound.set_volume(1)                     # Set volume to maximum
        victory_sound.play()
def snake_Sound():
        pygame.mixer.init()                          # Initialize pygame audio mixer
        snake_sound = pygame.mixer.Sound("snake_sound.mp3")  # Load bomb sound effect
        snake_sound.set_volume(1)                     # Set volume to maximum
        snake_sound.play()
def ladder_Sound():
        pygame.mixer.init()                          # Initialize pygame audio mixer
        ladder_sound = pygame.mixer.Sound("ladder_sound.mp3")  # Load bomb sound effect
        ladder_sound.set_volume(1)                     # Set volume to maximum
        ladder_sound.play()

def game_Sound():
        pygame.mixer.init()                          # Initialize pygame audio mixer
        game_sound = pygame.mixer.Sound("game_sound.mp3")  # Load bomb sound effect
        game_sound.set_volume(0.2)                     # Set volume to maximum
        game_sound.play(-1)
def draw_menu(selected_option):
    gameDisplay.fill(back)

    # Draw the title
    title_surface = pygame.font.SysFont("comicsansms", 70).render("Snakes and Ladders", True, black)
    title_rect = title_surface.get_rect(center=(display_width // 2, display_height // 5))
    gameDisplay.blit(title_surface, title_rect)

    y_pos = display_height // 2
    i = 0
    while i < len(MENU_OPTIONS):
        option = MENU_OPTIONS[i]
        color = darkback if i == selected_option else white
        text_surface = pygame.font.SysFont("comicsansms", 40).render(option, True, color)
        text_rect = text_surface.get_rect(center=(display_width // 2, y_pos))
        pygame.draw.rect(gameDisplay, black, text_rect.inflate(20, 10), border_radius=10)
        gameDisplay.blit(text_surface, text_rect)
        y_pos += 100
        i += 1

    pygame.display.flip()

def main_menu():
    selected_option = 0  # Initially, the first option is selected
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                # Navigate the menu using arrow keys
                if event.key == pygame.K_DOWN:
                    selected_option = (selected_option + 1) % len(MENU_OPTIONS)
                elif event.key == pygame.K_UP:
                    selected_option = (selected_option - 1) % len(MENU_OPTIONS)
                elif event.key == pygame.K_RETURN:
                    if MENU_OPTIONS[selected_option] == "Start Game":
                        return  # Exit the function to continue the game
                    elif MENU_OPTIONS[selected_option] == "Continue":
                        if saved_games():
                            return True
                        return
                    elif MENU_OPTIONS[selected_option] == "Quit":
                        pygame.quit()
                        sys.exit()

        draw_menu(selected_option)
def saved_games():
    global player1, player2
    try:
        load_game_state(f'{player1["name"]}_{player2["name"]}.txt')

        return True
    except FileNotFoundError:
        draw_text("No saved games found!", display_width // 2, display_height // 3, center=True)
        pygame.display.flip()
        pygame.time.wait(2000)  # Wait for 2 seconds
        return

def randint_exclude(start, end, exclude):
    result = exclude
    while result == exclude:
        result = random.randint(start, end)
    return result

def draw_text(text, x, y, color=black, center=False, size=36):
    render = pygame.font.Font(None, size).render(text, True, color)
    if center:
        rect = render.get_rect(center=(x, y))
        gameDisplay.blit(render, rect)
    else:
        gameDisplay.blit(render, (x, y))

def names_page():
    input_box1 = pygame.Rect(display_width // 2 - 100, display_height // 3, 200, 40)
    input_box2 = pygame.Rect(display_width // 2 - 100, display_height // 3 + 100, 200, 40)
    active_box = None
    player1_name = ""
    player2_name = ""

    while True:
        gameDisplay.fill(back)
        draw_text("Welcome", display_width // 2, display_height // 3 - 100, center=True, color=white, size=50)
        draw_text("Enter Player 1 Name or comp:", display_width // 2, display_height // 3 - 30, center=True)
        draw_text("Enter Player 2 Name or comp:", display_width // 2, display_height // 3 + 70, center=True)
        pygame.draw.rect(gameDisplay, forg if active_box == input_box1 else darkback, input_box1, 2)
        pygame.draw.rect(gameDisplay, forg if active_box == input_box2 else darkback, input_box2, 2)
        draw_text(player1_name, input_box1.x + 10, input_box1.y + 10)
        draw_text(player2_name, input_box2.x + 10, input_box2.y + 10)
        draw_text("Press Enter to Start", display_width // 2, display_height - 100, center=True)
        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                if input_box1.collidepoint(event.pos):
                    active_box = input_box1
                elif input_box2.collidepoint(event.pos):
                    active_box = input_box2
                else:
                    active_box = None
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN and player1_name and player2_name:
                    if player1_name == player2_name:
                        return player1_name, player2_name + '(2)'
                    return player1_name, player2_name                
                if active_box == input_box1:
                    if event.key == pygame.K_BACKSPACE:
                        player1_name = player1_name[:-1]
                    else:
                        player1_name += event.unicode
                elif active_box == input_box2:
                    if event.key == pygame.K_BACKSPACE:
                        player2_name = player2_name[:-1]
                    else:
                        player2_name += event.unicode


def run_decider_page():
    global roll1, roll2, player1, player2, textRect, textSurf, crashed
    turn_message = f"{player1['name']}, press Space to roll the dice"
    turn = 0

    while not crashed:
        gameDisplay.fill(back)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                crashed = True

            # If it's a human player's turn, check for SPACE key press
            elif event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                if roll1 is None:
                    if player1['name'] == 'syntax error':
                        roll1 = 1
                    elif player2["name"] == 'syntax error':
                        roll1 = random.randint(2,6)
                    else:
                        roll1 = random.randint(1, 6)
                    turn_message = f"{player2['name']}, press Space to roll the dice"
                elif roll2 is None:
                    if player2['name'] == 'syntax error':
                        roll2 = 1
                    else:
                        roll2 = randint_exclude(1, 6, roll1)

        # If it's the computer player's turn, roll automatically
        if (player1["name"] == 'comp' or player1["name"] == 'comp(2)') and roll1 is None:
            roll1 = random.randint(1, 6)
            turn_message = f"{player2['name']}, press Space to roll the dice"

        elif (player2["name"] == 'comp' or player2["name"] == 'comp(2)') and roll2 is None and roll1 is not None:
            roll2 = randint_exclude(1, 6, roll1)

        smallText = pygame.font.SysFont("comicsansms", 40)
        textSurf, textRect = text_objects(turn_message, smallText, darkback, back)
        textRect.center = (400, 300)
        gameDisplay.blit(textSurf, textRect)

        if roll1 is not None:
            textSurf, textRect = text_objects(f"{player1['name']} rolls: {roll1}", smallText, darkback, back)
            textRect.center = (400, 350)
            gameDisplay.blit(textSurf, textRect)

        if roll2 is not None:
            textSurf, textRect = text_objects(f"{player2['name']} rolls: {roll2}", smallText, darkback, back)
            textRect.center = (400, 400)
            gameDisplay.blit(textSurf, textRect)

        if roll1 is not None and roll2 is not None:
            if roll1 > roll2:
                textSurf, textRect = text_objects(f"{player1['name']} starts!", smallText, darkback, back)
                turn = 1
            else:
                textSurf, textRect = text_objects(f"{player2['name']} starts!", smallText, darkback, back)
                turn = 2
            textRect.center = (650, 450)
            gameDisplay.blit(textSurf, textRect)
            pygame.display.update()
            pygame.time.wait(2000)  # Display the rolls for 2 seconds
            return turn
        
        pygame.display.update()
# after Win
def WINNER():
    global player2, player1
    pygame.draw.rect(gameDisplay, back, (95 + 30, 95 + 30, 610, 410))
    pygame.draw.rect(gameDisplay, darkback, (100 + 30, 100 + 30, 600, 400))
    pygame.draw.rect(gameDisplay, back, (105 + 30, 105 + 30, 590, 390))
    Victory_Sound()
    smallText = pygame.font.SysFont("comicsansms", 90)
    textSurf, textRect = text_objects("GAME OVER", smallText, darkback)
    textRect.center = (display_width // 2, display_height // 2 - 100)
    gameDisplay.blit(textSurf, textRect)

    smallText = pygame.font.SysFont("comicsansms", 50)
    if turn == 1:
        textSurf, textRect = text_objects(f"WINNER: {player1['name']}", smallText, darkback)
    elif turn == 2:
        textSurf, textRect = text_objects(f"WINNER: {player2['name']}", smallText, darkback)
    textRect.center = (display_width // 4 + 220, display_height // 2 + 100)
    gameDisplay.blit(textSurf, textRect)

    temp = True
    while temp:
        global crashed, DONE1
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                crashed = True
                temp = False
                pygame.quit()
                sys.exit()

        pygame.display.update()

def initialize_player(boardarr, color):
    val = 0
    xpos, ypos = None, None
    size = random.randint(18, 22)
    for x, y, num in boardarr:
        if val == num:
            xpos = x
            ypos = y
    return {"val": val, "xpos": xpos, "ypos": ypos, "color": color, "size": size, 'old_val': 0}

def move_player(player, no, opponent):
    new_position = player["val"] + no  # Calculate the new position based on the dice roll
    if new_position > 100:
        return
    player["val"] = new_position
    update_player_position(player)
    # Check if the player reaches or exceeds square 100 (winning)
    if player['val'] == 100:
        global DONE1,crashed
        DONE1 ,crashed= True,True
        draw_player(player,opponent)
        WINNER()
        draw_player(player, opponent)
    elif player['val'] < 100:
        # Check if there's a collision between players on the same square
        if player['val'] == opponent["val"]:
            opponent['val'] = 0
            opponent['old_val'] = 0
            update_player_position(opponent)
            update_player_old_position(opponent)
    
def in_tuple(no, event):
    for i in event:
        if no in i:
            return True
    return False

def get_event(no):
    global snakes, ladders
    if in_tuple(no, snakes):
        for x, y in snakes:
            if y == no:
                return x
    elif in_tuple(no, ladders):
        for x, y in ladders:
            if y == no:
                return x
    else:
        return 0

def update_player_position(player):
    global board_map, snakes, ladders
    update_player_old_position(player)
    for x, y, num in board_map:
        if player['val'] == num:
            player['xpos'] = x
            player['ypos'] = y
    for x, y in ladders:
        if player['val'] == x:
            player['val'] = y
            update_player_position(player)
    for x, y in snakes:
        if player['val'] == x:
            player['val'] = y
            update_player_position(player)

def update_player_old_position(player):
    global board_map
    for x, y, num in board_map:
        if player['old_val'] == num:
            player['old_xpos'] = x
            player['old_ypos'] = y

def draw_player(player, opponent):
    update_player_old_position(player)
    if not (7 > player['val'] - player['old_val'] > -1) and player['val'] != 0:

        temp_val = get_event(player['val'])
        while player["old_val"] <= temp_val:
            pygame.draw.circle(gameDisplay, player["color"], (player["old_xpos"], player["old_ypos"]), player["size"])
            pygame.display.update()
            pygame.time.wait(400)
            gameDisplay.blit(board, board_rect)
            draw_player(opponent, player)
            pygame.display.update()
            player['old_val'] += 1
            update_player_old_position(player)
        if player['val']<player['old_val']:
            snake_Sound()
        else:
            ladder_Sound()
        pygame.draw.circle(gameDisplay, player["color"], (player["xpos"], player["ypos"]), player["size"])
        player['old_val'] = player['val']
        return

    while player["old_val"] < player['val']:
        pygame.draw.circle(gameDisplay, player["color"], (player["old_xpos"], player["old_ypos"]), player["size"])
        pygame.display.update()
        pygame.time.wait(200)
        gameDisplay.blit(board, board_rect)
        draw_player(opponent, player)
        pygame.display.update()
        player['old_val'] += 1
        update_player_old_position(player)
    pygame.draw.circle(gameDisplay, player["color"], (player["xpos"], player["ypos"]), player["size"])
    

def text_objects(text, font, clr, background=None):
    if background:
        textSurface = font.render(text, True, clr, background)
    else:
        textSurface = font.render(text, True, clr)
    return textSurface, textSurface.get_rect()

def save_game_state():
    with open(f"{player1['name']}_{player2['name']}.txt", "w") as file:
        file.write(f"{turn}\n")
        file.write(f"{player1['name']}\n {player1['val']}\n {player1['xpos']}\n {player1['ypos']}\n {player1['old_val']}\n")
        file.write(f"{player2['name']}\n {player2['val']}\n {player2['xpos']}\n {player2['ypos']}\n{player2['old_val']}\n")
        print("Game state saved.")

def save_popup():
    screen = pygame.Surface((400, 200))
    screen.fill(back)

    font = pygame.font.SysFont("comicsansms", 25)
    textSurf, textRect = text_objects("Do you want to save the game?", font, darkback)
    textRect.center = (200, 50)
    screen.blit(textSurf, textRect)

    yes_button = pygame.Rect(50, 120, 100, 50)
    no_button = pygame.Rect(250, 120, 100, 50)

    pygame.draw.rect(screen, darkback, yes_button)
    pygame.draw.rect(screen, darkback, no_button)

    textSurf, textRect = text_objects("Yes", font, back)
    textRect.center = yes_button.center
    screen.blit(textSurf, textRect)

    textSurf, textRect = text_objects("No", font, back)
    textRect.center = no_button.center
    screen.blit(textSurf, textRect)

    gameDisplay.blit(screen, (200, 200))
    pygame.display.flip()

    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return 
            if event.type == pygame.MOUSEBUTTONDOWN:
                mouse_x, mouse_y = event.pos
                adjusted_x = mouse_x - 200  # Adjust the x-coordinate
                adjusted_y = mouse_y - 200  # Adjust the y-coordinate
                if yes_button.collidepoint(adjusted_x, adjusted_y):
                    save_game_state()  # Call the save game function
                    return 
                elif no_button.collidepoint(adjusted_x, adjusted_y):
                    return 

def load_game_state(game_name):
    global turn, player1, player2, did_continue
    with open(game_name, "r") as file:
        lines = file.readlines()
        turn = int(lines[0].strip())

        # Read player data
        player1_data = lines[1:6]
        player2_data = lines[6:]
    
        did_continue = True

        player1_new = {
            "val": int(player1_data[1].strip()),
            "xpos": int(player1_data[2].strip()),
            "ypos": int(player1_data[3].strip()),
            "old_val":int(player1_data[4].strip())
        }
        player1.update(player1_new)
        
        player2_new = {
            "val": int(player2_data[1].strip()),
            "xpos": int(player2_data[2].strip()),
            "ypos": int(player2_data[3].strip()),
            "old_val":int(player2_data[4].strip())
        }
        player2.update(player2_new)


def dice(no):
    x, y, w, h = 635, 50, 190, 190
    pygame.draw.rect(gameDisplay, darkback, (x, y, w, h))
    if no == 1:
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 2)), (y + (h // 2))), size)
    elif no == 2:
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (h // 2))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (h // 2))), size)
    elif no == 3:
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (3 * h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 2)), (y + (h // 2))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (h // 4))), size)
    elif no == 4:
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (3 * h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (3 * h // 4))), size)
    elif no == 5:
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 2)), (y + (h // 2))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (3 * h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (h // 4))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (3 * h // 4))), size)
    elif no == 6:
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (h // 2))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (h // 2))), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (h // 4)) - 10), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (w // 4)), (y + (3 * h // 4)) + 10), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (h // 4)) - 10), size)
        pygame.draw.circle(gameDisplay, forg, ((x + (3 * w // 4)), (y + (3 * h // 4)) + 10), size)

def game_loop():
    global turn
    crashed = False
    DONE1 = False
    first_roll = False
    roll = False
    syntax_count = 0
    no=0
    old_no=[0,0]
    clock = pygame.time.Clock()
    game_Sound()

    while not crashed:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_popup()
                crashed = True
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    roll = not roll

        if not DONE1:
            if not first_roll:
                gameDisplay.fill(back)
                first_roll = True
                if turn == 1:
                    pygame.draw.circle(gameDisplay, player1["color"], (730, 400), 50)
                elif turn == 2:
                    pygame.draw.circle(gameDisplay, player2["color"], (730, 400), 50)
                gameDisplay.blit(board, board_rect)
            draw_player(player1, player2)
            draw_player(player2, player1)

            if ((player1["name"] == 'comp' or player1["name"] == 'comp(2)') and turn == 1) or (turn == 2 and (player2["name"] == 'comp' or player2["name"] == 'comp(2)')):
                roll = not roll
            smallText = pygame.font.SysFont("comicsansms", 25)
            pygame.draw.rect(gameDisplay, back, (615, 270, 300, 70))
            if turn == 1:
                textSurf, textRect = text_objects(f"{player1['name']}'s Turn", smallText, darkback, back)
                pygame.draw.circle(gameDisplay, player1["color"], (730, 400), 50)
            elif turn == 2:
                textSurf, textRect = text_objects(f"{player2['name']}'s Turn", smallText, darkback, back)
                pygame.draw.circle(gameDisplay, player2["color"], (730, 400), 50)
            textRect.center = (730, 300)
            gameDisplay.blit(textSurf, textRect)

            smallText1 = pygame.font.SysFont("comicsansms", 20)
            textSurf, textRect = text_objects("Hit Space to Play", smallText1, darkback, back)
            textRect.center = (730, 500)
            gameDisplay.blit(textSurf, textRect)

            if roll:
                old_no[turn-1]=no
                dice_sound()
                time = random.randint(10, 15)
                for _ in range(time):
                    no = random.randint(1, 6)
                    dice(no)
                    pygame.time.wait(100)
                    pygame.display.update()
                if player1['name'] == 'syntax error' and turn == 1 or player2['name'] == 'syntax error' and turn == 2:
                    no = syntax_error[syntax_count]
                    syntax_count += 1
                else:
                    no = random.randint(1, 6)
                dice(no)

                roll = False
                if turn == 1:
                    move_player(player1, no, player2)
                    turn = 1 if no==6 and old_no[0]!=6 else 2
                    
                elif turn == 2:
                    move_player(player2, no, player1)
                    turn = 2 if no==6 and old_no[1]!=6 else 1
                    


        pygame.display.update()
        clock.tick(60)

    pygame.quit()
    quit()

def main():
    global player1, player2, first_roll, roll1, roll2, turn
    pygame.init()
    pygame.mixer.init()
    pygame.mixer.music.load('welcome_sound.mp3')  # Load your music file
    pygame.mixer.music.play(-1)  # -1 means the music will loop indefinitely
    pygame.mixer.music.set_volume(0.6)
    player1 = initialize_player(board_map, pla1clr)
    player2 = initialize_player(board_map, pla2clr)
    gameDisplay.fill(back)

    # Initial dice roll to determine who starts
    name1, name2 = names_page()
    player1['name'], player2['name'] = name1, name2
    if main_menu(): # Call the main menu function 
        pygame.mixer.music.stop()
        game_loop()
    else:
        turn = run_decider_page()
        pygame.mixer.music.stop()
        game_loop()  # Call the game loop function

# Start the game
main()

