# HTTP for fun (250 points) - Network / Crypto

# Context:

HTTP for fun - This server may reveal the flag if you send the proper request

[https://c106.ctfsig.org/9bf0c855d9d39ecb92f819dcb532f422/](https://c106.ctfsig.org/9bf0c855d9d39ecb92f819dcb532f422/)

# Solution:

Accessing the site, we get the response of “You must provide the token to receive the flag”. 

![Figure 1.0 - Accessing the site](HTTP%20for%20fun%20(250%20points)%20-%20Network%20Crypto%2021eb0db6ff27806681afe55cd754fcd0/image.png)

Figure 1.0 - Accessing the site

While toying with some request in the website, one of my teammates noticed that when you change the HTTP request method, an unknown HTTP response header appears. In the image below, when changing “GET” to “PATCH”, we get a response header of X-R3. 

![Figure 1.1 - PATCH HTTP Request ](HTTP%20for%20fun%20(250%20points)%20-%20Network%20Crypto%2021eb0db6ff27806681afe55cd754fcd0/image%201.png)

Figure 1.1 - PATCH HTTP Request 

Trying all common HTTP request methods, we get the following:

For PATCH: X-R3: I F T
For PUT: X-Pb: QA YH SL EX DT PR ZV UM
For HEAD: X-R1: III K J
For DELETE: X-Msg: LEXJKFHVGSQQHRYIRPELAUIUCJH
For OPTIONS: X-Ref: ukw b

Additionally, when requesting POST, we get the response from below:

![Figure 1.2 - POST Request ](HTTP%20for%20fun%20(250%20points)%20-%20Network%20Crypto%2021eb0db6ff27806681afe55cd754fcd0/58cc6a10-cded-4fd4-9163-2050761e1878.png)

Figure 1.2 - POST Request 

Now, we can already assume to send the token as a JSON. 

Going back to the gathered headers, I noticed that it is the parameters for an Enigma Machine. 

Pb for Plug-board, R1 and R3 for Rotor 1 and Rotor 3 respectively, UKW B as the Reflector and msg for the encrypted message. At this point I already tried all known HTTP request methods, but I still have yet to get the settings for Rotor 2. Thus, I thought of bruteforcing Rotor 2 since the combinations for the settings for Rotor 2 for Enigma I is already countable and not relatively that many: 

$$
Possible ~Orders * Possible ~Rings * Possible ~Positions = Total ~Combinations
$$

Since there are 5 possible orders and I and III are already taken, and the possible rings and positions are just the English alphabet, so we get the following:

$$
(5 - 2) * 26 * 26 = 2028   ~combinations
$$

The following settings are known:

Message: LEXJKFHVGSQQHRYIRPELAUIUCJH
Plugboard: QA YH SL EX DT PR ZV UM
Rotor 1: III K J
Rotor 3: I F T
Reflector: UKW B

However, for this case, we do not know whether the second value and third value for the rotors are the positions or rings of the Enigma machine. So, in reality, there will be a total of 4056 combinations, but there will also be duplicates due to how an Enigma machine works. So, I just created a Python script that would automatically generate all these combinations.

```python
import string

# Rotor wirings
ROTOR_WIRINGS = {
    'I':   {'wiring': 'EKMFLGDQVZNTOWYHXUSPAIBRCJ', 'notch': 'Q'},
    'II':  {'wiring': 'AJDKSIRUXBLHWTMCQGZNPYFVOE', 'notch': 'E'},
    'III': {'wiring': 'BDFHJLCPRTXVZNYEIWGAKMUSQO', 'notch': 'V'},
    'IV':  {'wiring': 'ESOVPZJAYQUIRXHMCTNWDBKFLG', 'notch': 'J'},
    'V':   {'wiring': 'VZBRGITYUPSDNHLXAWMJQOFECK', 'notch': 'Z'},
}

# Reflector wirings (A-Z mapping for 0-25)
REFLECTOR_WIRINGS = {
    'UKW-B': 'YRUHQSLDPXNGOKMIEBFZCWVJAT',
}

ALPHABET = string.ascii_uppercase

# --- Helper Functions ---
def char_to_int(char):
    """Converts a character (A-Z) to its integer index (0-25)."""
    return ALPHABET.index(char.upper())

def int_to_char(integer):
    """Converts an integer index (0-25) to its character (A-Z)."""
    return ALPHABET[integer]

class Rotor:
    """Represents a single Enigma rotor."""

    def __init__(self, rotor_type, ring_setting_char, initial_position_char):
        if rotor_type not in ROTOR_WIRINGS:
            raise ValueError(f"Invalid rotor type: {rotor_type}")

        self.rotor_type = rotor_type
        self.wiring = ROTOR_WIRINGS[rotor_type]['wiring']
        self.notch = ROTOR_WIRINGS[rotor_type]['notch']
        
        # Ring setting: determines the position of the internal wiring relative to the ring.
        # It's set from 0-25 based on A-Z.
        # An incrementing ring_setting moves the internal wiring counter-clockwise.
        self.ring_setting = char_to_int(ring_setting_char) 

        # It changes as the rotor steps.
        self.position = char_to_int(initial_position_char)

    def _map(self, input_index, direction='forward'):
        """
        Maps an electrical signal through the rotor's wiring.
        
        input_index: The index (0-25) of the incoming signal relative to the stator (fixed part of machine).
        direction: 'forward' (entry to exit) or 'backward' (exit to entry).
        """
        # Adjust for current rotor position and ring setting
        # The signal enters relative to the fixed part of the machine.
        # We need to find its effective position relative to the rotor's internal wiring.
        offset_input = (input_index + self.position - self.ring_setting) % 26

        if direction == 'forward':
            output_char = self.wiring[offset_input]
        elif direction == 'backward':
            # To map backward, we need to find which input to the wiring leads to offset_input.
            # This is effectively finding the inverse mapping.
            # The signal comes back *from* the reflector, entering the rotor's "output" side.
            # `ALPHABET[offset_input]` is the character on the internal wiring's output pin.
            # We need to find which character on the internal wiring's *input* pin mapped to it.
            output_char = ALPHABET[self.wiring.index(ALPHABET[offset_input])]
        else:
            raise ValueError("Direction must be 'forward' or 'backward'")

        # Adjust back for rotor position and ring setting for the output signal
        # The output index from the internal wiring is adjusted back to the stator's reference.
        output_index = (char_to_int(output_char) - self.position + self.ring_setting) % 26
        return output_index

    def step(self):
        """Steps the rotor by one position."""
        self.position = (self.position + 1) % 26

    def at_notch(self):
        """
        Checks if the rotor is at its notch position.
        This is typically determined by the character visible in the window.
        The ring setting does not usually affect this check in standard Enigma I simulations,
        as the notch is a physical feature aligned with a specific character on the rotor's outer ring.
        """
        return self.position == char_to_int(self.notch)

class Reflector:
    """Represents the Enigma reflector."""

    def __init__(self, reflector_type):
        if reflector_type not in REFLECTOR_WIRINGS:
            raise ValueError(f"Invalid reflector type: {reflector_type}")
        self.wiring = REFLECTOR_WIRINGS[reflector_type]

    def reflect(self, input_index):
        """Reflects an electrical signal."""
        return char_to_int(self.wiring[input_index])

class Enigma:
    """Simulates the Enigma machine."""

    def __init__(self, rotor_settings, reflector_type, plugboard_settings):
        # rotor_settings: list of tuples (rotor_type, ring_setting_char, initial_position_char)
        # Order: Left Rotor (self.rotors[0]), Middle Rotor (self.rotors[1]), Right Rotor (self.rotors[2])
        self.rotors = [Rotor(*settings) for settings in rotor_settings]
        self.reflector = Reflector(reflector_type)
        self.plugboard = self._setup_plugboard(plugboard_settings)

    def _setup_plugboard(self, settings_string):
        """
        Sets up the plugboard mapping.
        settings_string: "QA YH SL EX DT PR ZV UM"
        """
        mapping = {char_to_int(char): char_to_int(char) for char in ALPHABET}
        pairs = settings_string.split()
        for pair in pairs:
            if len(pair) != 2:
                raise ValueError(f"Invalid plugboard pair: {pair}. Must be two characters.")
            char1, char2 = pair[0], pair[1]
            idx1, idx2 = char_to_int(char1), char_to_int(char2)
            mapping[idx1] = idx2
            mapping[idx2] = idx1
        return mapping

    def _step_rotors(self):
        """
        Performs rotor stepping according to Enigma I rules (double stepping).
        This method determines which rotors should step based on their *current* positions
        (before any stepping for the current key press) and then applies the steps.
        """
        # Determine stepping conditions based on current rotor positions
        # If R2 is at its notch, R1 will step, and R2 will step (double step).
        r1_will_step = self.rotors[1].at_notch()
        
        # If R3 is at its notch, R2 will step.
        r2_will_step_due_to_r3 = self.rotors[2].at_notch()

        # Step R3 (rightmost rotor) always
        self.rotors[2].step()

        # Step R2 (middle rotor) if R2 was at its notch (double step), OR if R3 was at its notch.
        if r1_will_step or r2_will_step_due_to_r3:
            self.rotors[1].step()
        
        # Step R1 (leftmost rotor) only if R2 was at its notch (double step).
        if r1_will_step:
            self.rotors[0].step()

    def encrypt_char(self, char):
        """Encrypts a single character."""
        # 1. Step rotors (this happens BEFORE the signal passes)
        self._step_rotors()

        # Convert character to index (0-25)
        input_index = char_to_int(char)

        # 2. Plugboard In
        input_index = self.plugboard[input_index]

        # 3. Rotors (Right to Left - Forward Pass)
        # Signal passes through R3, then R2, then R1.
        for i in range(2, -1, -1): # Iterate from rightmost (2) to leftmost (0)
            input_index = self.rotors[i]._map(input_index, direction='forward')

        # 4. Reflector
        input_index = self.reflector.reflect(input_index)

        # 5. Rotors (Left to Right - Backward Pass)
        # Signal passes through R1, then R2, then R3.
        for i in range(0, 3): # Iterate from leftmost (0) to rightmost (2)
            input_index = self.rotors[i]._map(input_index, direction='backward')

        # 6. Plugboard Out
        output_index = self.plugboard[input_index]

        return int_to_char(output_index)

    def encrypt_message(self, message):
        """Encrypts an entire message."""
        encrypted_message = []
        for char in message.upper():
            if char in ALPHABET:
                encrypted_message.append(self.encrypt_char(char))
            else:
                encrypted_message.append(char) # Keep non-alphabetic characters as is
        return "".join(encrypted_message)

if __name__ == "__main__":
    MESSAGE = "LEXJKFHVGSQQHRYIRPELAUIUCJH"
    PLUGBOARD = "QA YH SL EX DT PR ZV UM"
    REFLECTOR_TYPE = "UKW-B"

    # Fixed parameters for Rotor 1 and Rotor 3 as per your latest clarification:
    # Rotor 1 (leftmost): Type III, Ring K, Pos J
    # Rotor 3 (rightmost): Type I, Ring F, Pos T
    FIXED_ROTOR_1_TYPE = 'III'
    FIXED_ROTOR_1_RING = 'K'  # Ring setting for Rotor 1
    FIXED_ROTOR_1_POS = 'J'   # Initial position for Rotor 1

    FIXED_ROTOR_3_TYPE = 'I'
    FIXED_ROTOR_3_RING = 'F'  # Ring setting for Rotor 3
    FIXED_ROTOR_3_POS = 'T'   # Initial position for Rotor 3

    # Possible rotor types for Rotor 2: Cannot be the same as Rotor 1 or Rotor 3.
    POSSIBLE_ROTOR_2_TYPES = [r_type for r_type in ROTOR_WIRINGS if r_type not in [FIXED_ROTOR_1_TYPE, FIXED_ROTOR_3_TYPE]]

    print(f"Bruteforcing Rotor 2 for message: {MESSAGE}")
    print(f"Fixed Rotor 1: {FIXED_ROTOR_1_TYPE} (Ring={FIXED_ROTOR_1_RING}, Pos={FIXED_ROTOR_1_POS})")
    print(f"Fixed Rotor 3: {FIXED_ROTOR_3_TYPE} (Ring={FIXED_ROTOR_3_RING}, Pos={FIXED_ROTOR_3_POS})")
    print(f"Possible Rotor 2 types: {', '.join(POSSIBLE_ROTOR_2_TYPES)}")
    print("-" * 50)

    results_count = 0
    # Iterate through all possible Rotor 2 types (II, IV, V)
    for rotor2_type in POSSIBLE_ROTOR_2_TYPES: 
        # Iterate through all possible Ring Settings for Rotor 2 (A-Z)
        for rotor2_ring_char in ALPHABET: 
            # Iterate through all possible Initial Positions for Rotor 2 (A-Z)
            for rotor2_pos_char in ALPHABET: 
                results_count += 1
                # Define the full rotor configuration for the Enigma machine for this combination
                # Order: Left Rotor (R1), Middle Rotor (R2), Right Rotor (R3)
                rotor_settings = [
                    (FIXED_ROTOR_1_TYPE, FIXED_ROTOR_1_RING, FIXED_ROTOR_1_POS),
                    (rotor2_type, rotor2_ring_char, rotor2_pos_char),
                    (FIXED_ROTOR_3_TYPE, FIXED_ROTOR_3_RING, FIXED_ROTOR_3_POS)
                ]

                enigma = Enigma(rotor_settings, REFLECTOR_TYPE, PLUGBOARD)

                decrypted_message = enigma.encrypt_message(MESSAGE) 

                print(f"Rotor 2: Type={rotor2_type}, Ring={rotor2_ring_char}, Pos={rotor2_pos_char} -> Decrypted: {decrypted_message}")
    
    print("-" * 50)
    print(f"Bruteforce complete. Total combinations tested: {results_count}")

```

I was able to find the Token when the Rotor settings are the following:

Rotor 1: Type III, Ring K, Pos J
Rotor 2: Type V, Ring T, Pos M
Rotor 3: Type I, Ring F, Pos T

![Figure 1.3 - Token](HTTP%20for%20fun%20(250%20points)%20-%20Network%20Crypto%2021eb0db6ff27806681afe55cd754fcd0/image%202.png)

Figure 1.3 - Token

Knowing the the accepted request based on Figure 1.2, we can now send the request for the Token as a JSON object shows the flag in the response X-flag header. 

![Figure 1.4 - Flag!](HTTP%20for%20fun%20(250%20points)%20-%20Network%20Crypto%2021eb0db6ff27806681afe55cd754fcd0/image%203.png)

Figure 1.4 - Flag!