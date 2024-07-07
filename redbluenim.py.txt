import math

# Define a class for the Red Blue Nim game
class RedBlueNim:
    def __init__(self, piles, misere=False):
        self.piles = piles  # List of integers representing stone counts in each pile
        self.misere = misere  # Flag to indicate if it's misère version

    def is_game_over(self, maximizing_player):
        if self.misere:
            if maximizing_player:
                return all(pile == 0 for pile in self.piles)
            else:
                return sum(self.piles) == 1
        else:
            return all(pile == 0 for pile in self.piles)

    def legal_moves(self):
        moves = []
        for i in range(len(self.piles)):
            for j in range(1, self.piles[i] + 1):
                moves.append((i, j))
        return moves

    def make_move(self, move):
        pile, count = move
        self.piles[pile] -= count

    def undo_move(self, move):
        pile, count = move
        self.piles[pile] += count

    def evaluate_position(self, maximizing_player):
        if self.misere:
            if maximizing_player:
                return -sum(self.piles)
            else:
                return sum(self.piles)
        else:
            return sum(self.piles)

# Minimax algorithm with alpha-beta pruning
def minimax_alpha_beta(state, depth, alpha, beta, maximizing_player):
    if state.is_game_over(maximizing_player) or depth == 0:
        return state.evaluate_position(maximizing_player)

    if maximizing_player:
        max_eval = -math.inf
        for move in state.legal_moves():
            state.make_move(move)
            eval = minimax_alpha_beta(state, depth - 1, alpha, beta, False)
            state.undo_move(move)
            max_eval = max(max_eval, eval)
            alpha = max(alpha, eval)
            if beta <= alpha:
                break
        return max_eval
    else:
        min_eval = math.inf
        for move in state.legal_moves():
            state.make_move(move)
            eval = minimax_alpha_beta(state, depth - 1, alpha, beta, True)
            state.undo_move(move)
            min_eval = min(min_eval, eval)
            beta = min(beta, eval)
            if beta <= alpha:
                break
        return min_eval

# Example usage
def main():
    piles = [3, 4, 5]  # Initial stone counts in each pile
    misere = False  # Set to True for misère version, False for standard version
    game = RedBlueNim(piles, misere)

    while not game.is_game_over(True):  # True indicates it's the computer's turn (maximizing player)
        print(f"Current piles: {game.piles}")
        move = max(game.legal_moves(), key=lambda x: minimax_alpha_beta(game, 3, -math.inf, math.inf, True))
        game.make_move(move)
        print(f"Computer's move: Remove {move[1]} stones from pile {move[0]}")
        print()

        if game.is_game_over(True):
            break

        print(f"Current piles: {game.piles}")
        while True:
            try:
                pile = int(input("Enter pile index (0, 1, or 2): "))
                count = int(input(f"Enter count to remove (1 to {game.piles[pile]}): "))
                if 0 <= pile < len(game.piles) and 1 <= count <= game.piles[pile]:
                    break
                else:
                    print("Invalid input. Try again.")
            except ValueError:
                print("Invalid input. Try again.")
        
        move = (pile, count)
        game.make_move(move)
        print(f"Your move: Remove {move[1]} stones from pile {move[0]}")
        print()

    print("Game over!")
    print("Final piles:", game.piles)

    # Determine the winner based on the final state of the game
    if game.is_game_over(True):
        print("Computer wins!")
    else:
        print("You win!")

if __name__ == "__main__":
    main()