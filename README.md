import arcade
import numpy as np
import random
import json
import os

SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
SCREEN_TITLE = "ASMD - Educational Math Game"
SAVE_FILE = "asmd_progress.json"


class ASMD(arcade.Window):
    def __init__(self):
        super().__init__(SCREEN_WIDTH, SCREEN_HEIGHT, SCREEN_TITLE)

        # Game state
        self.level = 1
        self.score = 0
        self.highest_score = 0
        self.question_count = 0
        self.question = ""
        self.answer = 0
        self.input_text = ""
        self.game_over = False
        self.winner = False

        # Player info
        self.player_name = ""
        self.name_input_active = True
        self.name_input_text = ""

        # Load all player progress
        self.players_data = self.load_all_players()

        arcade.set_background_color(arcade.color.LIGHT_SKY_BLUE)

    # ------------------------------
    # SAVE / LOAD FUNCTIONS
    # ------------------------------
    def load_all_players(self):
        """Load all players from save file."""
        if os.path.exists(SAVE_FILE):
            with open(SAVE_FILE, "r") as f:
                return json.load(f)
        return {}

    def save_all_players(self):
        """Save all players data to file."""
        with open(SAVE_FILE, "w") as f:
            json.dump(self.players_data, f)

    def load_player_progress(self, name):
        """Load a specific player's progress."""
        if name in self.players_data:
            data = self.players_data[name]
            self.level = data.get("level", 1)
            self.highest_score = data.get("highest_score", 0)
        else:
            # New player starts fresh
            self.level = 1
            self.highest_score = 0

    def save_player_progress(self):
        """Save the current player's progress."""
        self.players_data[self.player_name] = {
            "level": self.level,
            "highest_score": self.highest_score
        }
        self.save_all_players()

    # ------------------------------
    # GAME LOGIC
    # ------------------------------
    def generate_question(self):
        if self.level <= 10:
            a, b = np.random.randint(1, 10, 2)
            ops = ['+', '-']
        elif self.level <= 20:
            a, b = np.random.randint(1, 20, 2)
            ops = ['+', '-', '*']
        elif self.level <= 40:
            a, b = np.random.randint(1, 50, 2)
            ops = ['+', '-', '*', '/']
        elif self.level <= 70:
            a, b = np.random.randint(10, 100, 2)
            ops = ['+', '-', '*', '/']
        else:
            a, b = np.random.randint(50, 200, 2)
            ops = ['+', '-', '*', '/']

        op = random.choice(ops)

        if op == '/':
            b = random.randint(1, 9)
            a = b * random.randint(1, 9)
            self.answer = a // b
            self.question = f"{a} ÷ {b} = ?"
        elif op == '+':
            self.answer = a + b
            self.question = f"{a} + {b} = ?"
        elif op == '-':
            self.answer = a - b
            self.question = f"{a} - {b} = ?"
        elif op == '*':
            self.answer = a * b
            self.question = f"{a} × {b} = ?"

        self.input_text = ""

    # ------------------------------
    # DRAW
    # ------------------------------
    def on_draw(self):
        self.clear()

        # Name input screen
        if self.name_input_active:
            arcade.draw_text("Enter  Player Name:", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2 + 50,
                             arcade.color.BLACK, 24, anchor_x="center")
            arcade.draw_text(self.name_input_text, SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2,
                             arcade.color.DARK_RED, 22, anchor_x="center")
            arcade.draw_text("(Press ENTER to confirm, letters only)", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2 - 50,
                             arcade.color.DARK_GRAY, 16, anchor_x="center")
            return

        # Title
        arcade.draw_text("🧮 ASMD 🧮", SCREEN_WIDTH / 2, SCREEN_HEIGHT - 60,
                         arcade.color.DARK_BLUE, 30, anchor_x="center")

        # Winner / Game Over messages
        if self.winner:
            arcade.draw_text("🎉 YOU COMPLETED ALL 100 LEVELS! 🎉", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2,
                             arcade.color.GREEN, 24, anchor_x="center")
            arcade.draw_text("Press ENTER to restart", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2 - 60,
                             arcade.color.BLACK, 16, anchor_x="center")
            return

        if self.game_over:
            arcade.draw_text("❌ Game Over ❌", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2,
                             arcade.color.RED, 30, anchor_x="center")
            arcade.draw_text("Press ENTER to restart", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2 - 60,
                             arcade.color.BLACK, 16, anchor_x="center")
            return

        # Player info top-right
        arcade.draw_text(f"Player: {self.player_name}", SCREEN_WIDTH - 200, SCREEN_HEIGHT - 80,
                         arcade.color.DARK_GREEN, 18)
        arcade.draw_text(f"Level: {self.level}/100", SCREEN_WIDTH - 200, SCREEN_HEIGHT - 110,
                         arcade.color.BLACK, 18)

        # Score info
        arcade.draw_text(f"Score: {self.score}", 50, SCREEN_HEIGHT - 80, arcade.color.BLACK, 18)
        arcade.draw_text(f"Highest Score: {self.highest_score}", 50, SCREEN_HEIGHT - 110,
                         arcade.color.BLUE, 16)
        arcade.draw_text(f"Question {self.question_count + 1}/8", SCREEN_WIDTH / 2, SCREEN_HEIGHT - 120,
                         arcade.color.DARK_RED, 18, anchor_x="center")

        # Question
        arcade.draw_text(f"Question: {self.question}", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2 + 80,
                         arcade.color.BLACK, 26, anchor_x="center")
        arcade.draw_text("Your Answer:", SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2 + 30,
                         arcade.color.BLACK, 20, anchor_x="center")
        arcade.draw_text(self.input_text, SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2,
                         arcade.color.DARK_RED, 22, anchor_x="center")

        arcade.draw_text("Press ENTER to submit, BACKSPACE to delete",
                         SCREEN_WIDTH / 2, 60, arcade.color.DARK_GRAY, 14, anchor_x="center")

    # ------------------------------
    # KEY HANDLING
    # ------------------------------
    def on_key_press(self, key, modifiers):
        if self.name_input_active:
            if key == arcade.key.ENTER:
                name = self.name_input_text.strip()
                if name.isalpha():  # Only letters allowed
                    self.player_name = name
                    self.name_input_active = False
                    self.load_player_progress(self.player_name)
                    self.generate_question()
            elif key == arcade.key.BACKSPACE:
                self.name_input_text = self.name_input_text[:-1]
            elif arcade.key.A <= key <= arcade.key.Z or key == arcade.key.SPACE:
                self.name_input_text += chr(key)
            return

        if self.winner or self.game_over:
            if key == arcade.key.ENTER:
                self.reset_game()
            return

        if arcade.key.KEY_0 <= key <= arcade.key.KEY_9:
            self.input_text += str(key - arcade.key.KEY_0)
        elif key == arcade.key.MINUS and len(self.input_text) == 0:
            self.input_text = "-"
        elif key == arcade.key.BACKSPACE:
            self.input_text = self.input_text[:-1]
        elif key == arcade.key.ENTER:
            self.check_answer()

    # ------------------------------
    # SCORING LOGIC
    # ------------------------------
    def check_answer(self):
        try:
            user_answer = int(self.input_text)
        except ValueError:
            self.generate_question()
            return

        if user_answer == self.answer:
            self.score += 2
        else:
            self.score -= 1

        self.question_count += 1
        if self.score > self.highest_score:
            self.highest_score = self.score

        if self.question_count >= 8:
            self.evaluate_level()
        else:
            self.generate_question()

    def evaluate_level(self):
        if self.score >= 10:
            self.next_level()
        else:
            self.game_over = True

    def next_level(self):
        self.level += 1
        self.question_count = 0
        self.score = 0

        if self.level > 100:
            self.winner = True
        else:
            self.save_player_progress()  # Save progress per player
            self.generate_question()

    def reset_game(self):
        self.score = 0
        self.question_count = 0
        self.input_text = ""
        self.game_over = False
        self.winner = False
        self.generate_question()


def main():
    window = ASMD()
    arcade.run()


if __name__ == "__main__":
    main()
