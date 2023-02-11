# Introduction

We were tasked with implementing a robust protocol for handling a distributed version of the Bingo game. Our goal was to create a secure version of the game in which no single entity had complete control and in which players could detect if the caller or any other player had cheated. To do this, we created a server (caller) and multiple clients (players) that communicated over a network and implemented features such as authentication and authorization, card generation and commitment, and a log of all messages and actions. We carefully designed and implemented these features to ensure the integrity and confidentiality of the game. The final product was a secure and fair version of the Bingo game that could be played over a network.

The game's features include:

- Authentication and authorization: Upon connection to the playing area, both players and the caller must be authenticated and authorized. Players will be identified by a sequence number, a public key, and a nickname, while the caller will be identified by their sequence number and public key. The playing area will have a list of citizens that can act as callers and will use digital signatures and password-based message authentication codes to ensure the commitment of each user's public key and nickname.

- Card generation and commitment: Each player will generate their own card with unique numbers and commit the card to the playing area using a digital signature. The card will be protected by this signature to ensure that it cannot be tampered with.

- Log of messages and actions: The playing area will keep a log of all messages and actions that have taken place in the game. The log will consist of entries with the following format: sequence, timestamp, hash(prev_entry), text, signature. Players and the caller can request this log and audit the events to ensure that the game is being played fairly.

Overall, these features work together to ensure the security and fairness of the game by providing mechanisms for authentication and authorization, protecting the integrity of the cards, and keeping a record of all actions taken in the game.

# Design

The TCP protocol is used in the communication protocol for the secure Bingo game to establish a reliable connection between the server (caller) and clients (players). The server listens for incoming connections from clients and establishes a socket for each client to communicate over. The clients also create sockets to connect to the server and exchange data with it.

To ensure the reliability of the connection, the TCP protocol uses sequence numbers and acknowledgement messages to keep track of the data being transmitted. If a packet is lost or corrupted, the receiver can request that it be retransmitted. This ensures that all data is delivered correctly and in the correct order.

In the context of the secure Bingo game, the TCP protocol is used to exchange messages and data related to the game, such as player and caller authentication and authorization, card generation and commitment, and the log of messages and actions. It is important to use a reliable protocol like TCP to ensure that all data is transmitted correctly and securely.

To authenticate and authorize players and the caller, the following steps are taken:

1. Upon connecting to the playing area, each user must provide their citizen card and password to authenticate their identity.
2. The playing area verifies the user's identity using the PKCS #11 module of the citizen card and the provided password.
3. If the user's identity is successfully verified, the playing area generates an asymmetric key pair for the user to use in the game. The private key is stored on the user's citizen card, while the public key is sent to the playing area.
4. The playing area signs the user's public key, nickname, and sequence number to create a commitment and sends this commitment back to the user.
5. The user verifies the commitment and sends it back to the playing area, along with a password-based message authentication code (MAC) of the commitment.
6. The playing area verifies the MAC and, if it is valid, adds the user to the list of authorized players or callers.

This process ensures that only authenticated and authorized users can participate in the game and prevents unauthorized users from tampering with the game. It also ensures the integrity of the user's public key and nickname, as they are signed and verified by the playing area.

To generate and commit the card, the following steps are taken:

1. Each player generates a list of unique numbers for their card. The number of numbers in the list should be equal to the number of positions on the card.
2. The player signs the list of numbers using the `sign` method in the `RSA` class. The signature is used to ensure the integrity of the card and prevent tampering.
3. The player sends the list of numbers and the signature to the playing area.
4. The playing area verifies the signature using the `verify` method in the `RSA` class and, if it is valid, stores the list of numbers as the player's card.

This process ensures that each player has a unique and tamper-proof card that can be used in the game. It also allows the playing area to keep a record of all the cards committed by the players.

To shuffle and sign the deck, the following steps are taken:

1. The caller generates a list of unique numbers for the deck and shuffles it randomly.
2. The caller signs the shuffled deck using the `sign` method in the `RSA` class. The signature is used to ensure the integrity of the deck and prevent tampering.
3. The caller sends the shuffled deck and signature to all the players.
4. Each player verifies the signature using the `verify` method in the `RSA` class and, if it is valid, shuffles the deck again to ensure that it is truly random.
5. Each player signs the shuffled deck using the `sign` method in the `RSA` class and sends the signature back to the caller.
6. The caller verifies the signatures from all the players using the `verify` method in the `RSA` class. If all the signatures are valid, the caller declares the deck to be shuffled and signed.

This process ensures that the deck is shuffled randomly and that its integrity is preserved. It also allows all the players to agree on the shuffled and signed deck, which is necessary for determining the winner(s) of the game.

# Protocol

Here follows the a summed up version of the main message types described in the Protocol with **P** as Player, **PA** as Playing Area and **C** as Caller:

| Message Type             | Description                                                                       |
| ------------------------ | --------------------------------------------------------------------------------- |
| "join_caller_request"    | Sent by the a user that wants to be a C                                          |
| "join_caller_response"   | Sent by the PA to confirm registration of a C.                                    |
| join_request             | Sent by the a user that wants to be a P                                          |
| join_response            | Sent by the server to confirm registration of a P.                                |
| "start_game"             | Sent by the C to start the game.                                                  |
| "send_card""             | The P will share their card                                                       |
| "validate_cards_success" | All the cards were validated successfully by a P or C                              |
| "validate_cards_error"   | All the C were not validated successfully by a P or C                              |
| "generate_deck_request"  | Request made to the C by the PA to generate the starting deck                     |
| "generate_deck_response" | The C reponds to the PA with the generated deck cyphered and shuffled              |
| "shuffle_request"        | Sent by the PA to as a player to shuffle and encrypt the deck                     |
| "shuffle_response"       | Response to the previous request.                                                 |
| "validate_decks"         | All the Ps and the C receive the deck and start validating all the generate decks |
| "choose_winner"          | Request made by the PA to ask all the Ps and the C to choose a winner             |
| "send_log"               | Request to send all the log until now                                             |

`Message` acts as a blueprint for different types of messages that can be sent between different players and/or the server.

Each message has a specific structure, as defined by its respective method, which returns a dictionary with specific key-value pairs. The message type is defined by the key "type", which is a string indicating the purpose of the message.

For example, the method `announce_winner` returns a message with "type": "announce_winner", and the winner's name is given by the key-value pair "winner".

The `send_message` decorator adds a header to the message before it is sent, allowing the recipient to identify and process the message.

Overall, the `Message` class provides a way to send and receive structured messages between the players and the server to facilitate the flow of information in the game.

# Authentication

We implemented a `CC` class for interacting with a Citizen Card, a Portuguese national ID card, through the use of the PyKCS11 library. The Citizen Card has a smart card chip which stores various types of data, including a certificate, public key and private key. The `CC` class has methods to obtain the certificate data, public key, and sign and verify messages. The class requires a PIN number as an argument when an instance of it is created.

The methods `get_cc_cert`, `get_cc_public_key`, `sign_message`, and `verify_signature` allow the program to interact with the Citizen Card. The method `get_cc_cert` retrieves the certificate data from the smart card and returns it as a hexadecimal string representation of the PEM encoded certificate. The method `get_cc_public_key` retrieves the public key from the smart card and returns it as a hexadecimal string representation of the PEM encoded public key. The method `sign_message` takes a message as an argument and signs the message using the private key stored on the smart card and returns the signature as a hexadecimal string representation of the signature data. The method `verify_signature` verifies the signature of a message using the given public key, message, and signature and returns a Boolean indicating whether the signature is valid or not.

This makes it so that the program is able to retrieve data from the Citizen Card and use it to perform signing and verifying operations on messages, also allowing to authenticate and authorize users, utilizing the cryptography and PyKCS11 libraries to interact with the smart card.

# Cheating

We implemented three types of cheating, each symbolized by its own flag: one where a user randomly selects a winner instead of the real one (`winner_cheat`), another where one would use a random RSA private key (`RSA_cheat` ) and at last one where one would use a random symmetric AES key instead (`AES_cheat`).

As previously stated, these are taken as flags in the user's constructors, as such:

```python
def __init__(self, host, port, name, rsa_cheat, aes_cheat, winner_cheat, pin, card_size):
```

And are used throughout the program using a simple probabilistic determination.

For example, in the case where we want to cheat on the correct winner:

```python
    def choose_winner(self, conn, data):
        self.check_signature(data)
        self.logger.info(f"Choosing winner")
        if random.randint(1, 100) > self.winner_cheat:
            winner = Game.winner(data["deck"], data["cards"])
        else:
            self.logger.info(f"CHEATING Winner Choice")
            winner = self.name
        self.logger.info(f"I decided that the winner is {winner}")
        Protocol.choose_winner_response(conn, self.randomize_private_key(), winner)
```
