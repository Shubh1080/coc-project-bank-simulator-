/* hangman.c
   Simple Hangman game for learning C concepts:
   - char arrays, for/while loops, if/else
   - functions for checking guesses
   - scanf/printf for I/O
*/

#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define LIVES_INITIAL 6

/* Check the guess against secret_word.
   If found, update display_word and return 1 (found).
   If not found, return 0 (not found).
   len is the length of the secret_word (and display_word).
*/
int check_and_update_guess(char guess, const char secret_word[], char display_word[], int len) {
    int found = 0;
    for (int i = 0; i < len; ++i) {
        if (secret_word[i] == guess) {
            if (display_word[i] != guess) { // only update if not already revealed
                display_word[i] = guess;
            }
            found = 1;
        }
    }
    return found;
}

/* Return 1 if display_word contains no underscores (player has won). */
int is_word_complete(const char display_word[], int len) {
    for (int i = 0; i < len; ++i) {
        if (display_word[i] == '_') return 0;
    }
    return 1;
}

int main(void) {
    /* Hard-coded secret word (uppercase). Change this to try other words. */
    const char secret_word[] = "MENTOR";
    int len = strlen(secret_word);

    /* Create display_word with same size, initialized to underscores and terminated with '\0'. */
    char display_word[100]; // plenty of room
    for (int i = 0; i < len; ++i) display_word[i] = '_';
    display_word[len] = '\0';

    int lives = LIVES_INITIAL;
    char guess;
    char guessed_letters[26 + 1]; // track guessed letters (A-Z)
    int guessed_count = 0;

    printf("Welcome to Hangman!\n");
    printf("You have %d lives. Guess the %d-letter word, one letter at a time.\n\n", lives, len);

    /* Main game loop: continue while player still has lives and hasn't won */
    while (lives > 0 && !is_word_complete(display_word, len)) {
        /* Print player's view with spaces between letters */
        printf("Word: ");
        for (int i = 0; i < len; ++i) {
            printf("%c ", display_word[i]);
        }
        printf("\n");

        printf("Lives left: %d\n", lives);

        /* Print letters already guessed (if any) */
        if (guessed_count > 0) {
            printf("Guessed letters: ");
            for (int i = 0; i < guessed_count; ++i) putchar(guessed_letters[i]);
            printf("\n");
        }

        printf("Enter your guess (single letter): ");
        if (scanf(" %c", &guess) != 1) {
            /* Input failure, clear and continue */
            int ch;
            while ((ch = getchar()) != '\n' && ch != EOF) {}
            printf("Invalid input. Try again.\n\n");
            continue;
        }

        /* Normalize guess to uppercase */
        guess = (char) toupper((unsigned char)guess);

        /* Validate guess is an alphabet letter */
        if (!isalpha((unsigned char)guess)) {
            printf("Please enter a valid alphabet letter.\n\n");
            continue;
        }

        /* Check if letter was already guessed */
        int already_guessed = 0;
        for (int i = 0; i < guessed_count; ++i) {
            if (guessed_letters[i] == guess) {
                already_guessed = 1;
                break;
            }
        }

        if (already_guessed) {
            printf("You already guessed '%c'. Try a different letter.\n\n", guess);
            continue; /* do not penalize */
        }

        /* Record guessed letter */
        if (guessed_count < (int)sizeof(guessed_letters) - 1) {
            guessed_letters[guessed_count++] = guess;
            guessed_letters[guessed_count] = '\0';
        }

        /* Check and update */
        int found = check_and_update_guess(guess, secret_word, display_word, len);
        if (found) {
            printf("Nice! The letter '%c' is in the word.\n\n", guess);
        } else {
            lives--;
            printf("Sorry, '%c' is not in the word. You lost 1 life.\n\n", guess);
        }
    }

    /* Game finished: determine win or loss */
    if (is_word_complete(display_word, len)) {
        printf("Congratulations! You won!\n");
        printf("The word was: %s\n", secret_word);
    } else {
        printf("You lost! The word was: %s\n", secret_word);
    }

    return 0;
}
