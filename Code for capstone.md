SLR Parsing Algorithm Implementation

from collections import defaultdict

class SLRParser:
    def __init__(self, grammar):
        self.grammar = grammar
        self.states = []
        self.parse_table = defaultdict(dict)

    def closure(self, items):
        closure_set = set(items)
        changed = True
        while changed:
            changed = False
            for item in list(closure_set):
                lhs, rest = item.split("->")
                if rest and rest[0].isupper():  # If the next symbol is a non-terminal
                    for production in self.grammar.get(rest[0], []):
                        new_item = f"{rest[0]}->{'.' + production}"
                        if new_item not in closure_set:
                            closure_set.add(new_item)
                            changed = True
        return closure_set

    def goto(self, state, symbol):
        next_state = set()
        for item in state:
            lhs, rest = item.split("->")
            dot_pos = rest.find('.')
            if dot_pos != -1 and dot_pos + 1 < len(rest) and rest[dot_pos + 1] == symbol:
                moved_item = lhs + "->" + rest[:dot_pos] + symbol + "." + rest[dot_pos + 2:]
                next_state |= self.closure([moved_item])
        return frozenset(next_state)

    def construct_states(self):
        start_item = f"S'->.S"
        initial_state = self.closure([start_item])
        self.states.append(initial_state)
        worklist = [initial_state]
        while worklist:
            state = worklist.pop()
            for symbol in self.grammar.keys():
                next_state = self.goto(state, symbol)
                if next_state and next_state not in self.states:
                    self.states.append(next_state)
                    worklist.append(next_state)

    def construct_parse_table(self):
        for index, state in enumerate(self.states):
            for item in state:
                lhs, rest = item.split("->")
                if rest.endswith('.'):
                    if lhs == "S'":
                        self.parse_table[index]['$'] = "accept"
                    else:
                        for terminal in self.follow(lhs):
                            self.parse_table[index][terminal] = f"reduce {lhs}->{rest[:-1]}"
                else:
                    dot_pos = rest.find('.')
                    next_symbol = rest[dot_pos + 1] if dot_pos + 1 < len(rest) else None
                    if next_symbol and next_symbol.islower():
                        next_state = self.goto(state, next_symbol)
                        if next_state in self.states:
                            self.parse_table[index][next_symbol] = f"shift {self.states.index(next_state)}"

    def follow(self, non_terminal):
        pass
