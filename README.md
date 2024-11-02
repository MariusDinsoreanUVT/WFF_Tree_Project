import itertools
import random



def AtomicFormula(ch):
    return ord(ch) <= 65 + 26 and ord(ch) >= 65
def SquareConnective(ch):
    return ch == "∨" or ch == "⇒" or ch == "∧" or ch == "⇔"

def is_and(ch):
    return ch == "∧"
def is_or(ch):
    return ch == "∨"
def is_implication(ch):
    return ch == "⇒"
def is_equivalence(ch):
    return ch == "⇔"
def is_negation(ch):
    return ch == "¬"

def implication(x, y):
    if x == True and y == True:
        return True
    if x == True and y == False:
        return False
    if x == False and y == True:
        return True
    if x == False and y == False:
        return True
def or_con(x, y):
    if x == True and y == True:
        return True
    if x == True and y == False:
        return True
    if x == False and y == True:
        return True
    if x == False and y == False:
        return False
def and_con(x, y):
    if x == True and y == True:
        return True
    if x == True and y == False:
        return False
    if x == False and y == True:
        return False
    if x == False and y == False:
        return False
def equivalence(x, y):
    if x == True and y == True:
        return True
    if x == True and y == False:
        return False
    if x == False and y == True:
        return False
    if x == False and y == False:
        return True
def neg(x):
    if x == True:
        return False
    else:
        return True


def Generate_All_Interpretations(n):
    all_interpretations = list(itertools.product([True, False], repeat=n))
    return all_interpretations

class Node:
    def __init__(self, value = None, parent = None, nrnodes = 0, truth = False):
        self.value = value
        self.left = None
        self.right = None
        self.nrnodes = nrnodes
        self.parent = parent
        self.truth = False

class Tree:
    def __init__(self):
        self.root = None
        self.leafs = []
        self.formulas = []
        self.current_node = None

    def build_tree(self, WFF):
        nr = 0
        for char in WFF:
            if char == "(":
                if self.root == None:
                    self.root = Node(nrnodes = 0)
                    self.current_node = self.root
                    self.current_node.left = Node(parent = self.current_node)
                    self.current_node.right = Node(parent = self.current_node)
                    self.current_node = self.current_node.left
                else:
                    self.current_node.left = Node(parent = self.current_node)
                    self.current_node.right = Node(parent = self.current_node)
                    self.current_node = self.current_node.left
            elif char == ")":
                if self.current_node and self.current_node.parent:
                    self.current_node = self.current_node.parent
            elif is_negation(char):
                self.current_node.parent.value = char
                if self.current_node.parent.right is not None:
                    self.current_node.parent.right = None
            elif AtomicFormula(char):
                if self.current_node == None:
                    self.current_node = Node(value = char)
                else:
                    self.current_node.value = char
                self.leafs.append(self.current_node)
                self.current_node = self.current_node.parent
            elif SquareConnective(char):
                self.current_node.value = char
                self.current_node = self.current_node.right
            print("step ->", nr, end=" ")
            self.Print_Tree(self.root)
            print()
            nr += 1

    def VerifyIfComplete(self, node):
        if node != None:
            self.root.nrnodes += 1
            if node.left is not None:
                self.VerifyIfComplete(node.left)
            if node.right is not None:
                self.VerifyIfComplete(node.right)

    def Print_Tree(self, node):
        if node != None:
            if is_negation(node.value) == True:
                if node.left != None:
                    print("(", end = "")
                    print(node.value, end = "")
                    self.Print_Tree(node.left)
                    print(")", end = "")
            else:
                if node.left is not None:
                    print("(", end = "")
                    self.Print_Tree(node.left)
                print(node.value, end = "")
                if node.right is not None:
                    self.Print_Tree(node.right)
                    print(")", end = "")

    def Solve_Interpretation(self, node):
        if node != None:
            if is_negation(node.value) == True:
                return neg(self.Solve_Interpretation(node.left))
            elif is_or(node.value) == True:
                return or_con(self.Solve_Interpretation(node.left), self.Solve_Interpretation(node.right))
            elif is_and(node.value) == True:
                return and_con(self.Solve_Interpretation(node.left), self.Solve_Interpretation(node.right))
            elif is_implication(node.value) == True:
                return implication(self.Solve_Interpretation(node.left), self.Solve_Interpretation(node.right))
            elif is_equivalence(node.value) == True:
                return equivalence(self.Solve_Interpretation(node.left), self.Solve_Interpretation(node.right))
            elif AtomicFormula(node.value) == True:
                return node.truth

    def Print_Every_Formula(self, node):
        if node != None:
            self.Print_Every_Formula(node.left)
            self.Print_Every_Formula(node.right)
            if is_negation(node.value) == True or SquareConnective(node.value) == True:
                self.formulas.append(node)

WFF = str(input())
while WFF != "*":
    op = 0
    cp = 0
    con = 0
    length = 0
    for char in WFF:
        if is_negation(char) == True or SquareConnective(char) == True or AtomicFormula(char) == True:
            length += 1
        if char == "(":
            op += 1
        elif char == ")":
            cp += 1
        elif is_negation(char) == True or SquareConnective(char) == True:
            con += 1
    nrchar = 0
    WFF_Tree = Tree()
    WFF_Tree.build_tree(WFF)
    WFF_Tree.VerifyIfComplete(WFF_Tree.root)
    if WFF_Tree.root is None:
        WFF_Tree.root = Node()
    # print(op, cp, con, length, WFF_Tree.root.nrnodes)
    if op == cp and con == op and length == WFF_Tree.root.nrnodes:
        print("It is a well formed propositional formula")

        used = dict()
        nr_diff_liefs = 0
        for i in WFF_Tree.leafs:
            if i.value not in used:
                WFF_Tree.formulas.append(i)
                used[i.value] = True
                nr_diff_liefs += 1

        WFF_Tree.Print_Every_Formula(WFF_Tree.root)
        All_Int = Generate_All_Interpretations(nr_diff_liefs)
        Nr_True_Int = 0
        Nr_All_Int = 2 ** nr_diff_liefs

        for Int in All_Int:
            I = dict()
            nr = 0
            for i in used:
                I[i] = Int[nr]
                nr += 1
            for i in WFF_Tree.leafs:
                i.truth = I[i.value]
            Nr_True_Int += WFF_Tree.Solve_Interpretation(WFF_Tree.root)
        if Nr_True_Int < Nr_All_Int:
            print("The proposition is invalid")
        elif Nr_All_Int == Nr_True_Int:
            print("The proposition is valid")
        if Nr_True_Int >= 1:
            print("The proposition is satisfiable")
        elif Nr_True_Int == 0:
            print("The proposition is unsatisfiable")

        print("|     ", end = "")
        for i in WFF_Tree.formulas:
            WFF_Tree.Print_Tree(i)
            if AtomicFormula(i.value) == True:
                print("     |     ", end = "")
            else:
                print(" | ", end="")
        print()
        for row in range(Nr_All_Int):
            print("|   ", end = "")
            for nr in range(0, nr_diff_liefs):
                Int = All_Int[row]
                for i in WFF_Tree.leafs:
                    if i.value == WFF_Tree.formulas[nr].value:
                        i.truth = Int[nr]
                if Int[nr] == True:
                    print(Int[nr], "   |   ", end = "")
                else:
                    print(Int[nr], "  |   ", end = "")
            for col in range(nr_diff_liefs, len(WFF_Tree.formulas)):
                x = WFF_Tree.Solve_Interpretation(WFF_Tree.formulas[col])
                if x == True:
                    print(x, "   |   ", end="")
                else:
                    print(x, "  |   ", end="")
            print()


    else:
        print("It is not a Well Formed Propositional Formula")
    WFF = str(input())

# Homework 2
# (((P ⇒ Q) ∨ S) ⇔ T)
# ((P ⇒ (Q ∧ (S ⇒ T))))
# (¬(B(¬Q)) ∧ R)
# ((P⇒Q)∧((¬Q)∧P))
# ((P⇒Q)⇒(Q⇒P))
# ((¬(P ∨ Q)) ∧ (¬Q))
# Homework 3
# ((P ⇒ Q) ∧ ((¬Q) ∧ (¬P)))
# ((P ⇒ Q) ⇒ ((Q ⇒ S) ⇒ ((P ∨ Q) ⇒ R)))
# ((¬(P ⇒ Q)) ⇔ ((P ∨ R) ∧ ((¬P) ⇒ Q)))
# ((P ⇔ Q) ⇔ (¬(P ⇒ (¬Q))))
# ex 1 - (P⇒((¬(¬(¬(¬(¬B)))))⇔(Q∧S))) (F ∨ G)
