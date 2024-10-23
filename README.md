Step 1: Data Structure
The core data structure will be a Node to represent the AST. We will define a class that holds
information about whether the node is an operator or operand and its left/right child nodes. Python
class Node:
def __init__(self, node_type, value=None, left=None, right=None): """ Represents a node in the AST. :param node_type: 'operator' for AND/OR or 'operand' for conditions. :param value: Value for operand nodes, None for operators. :param left: Left child node (if applicable). :param right: Right child node (if applicable). """
self.type = node_type # 'operator' or 'operand' self.value = value # e.g., age > 30
self.left = left # Left child node
self.right = right # Right child node
def __repr__(self):
if self.type == 'operand':
return f"Operand({self.value})" else:
return f"Operator({self.value})" Step 2: Rule Creation (AST Construction)
To build the AST from a rule string, we'll parse the input string and create corresponding Node objects. We'll use recursive parsing to break the string into smaller sub-expressions. python
import re
def create_rule(rule_string): """ Parse the rule string into an AST. :param rule_string: String representing the rule. :return: Root node of the AST. """ # Tokenize the rule
tokens = re.findall(r'\(|\)|AND|OR|>|<|=|\'[^\']*\'|\d+|[a-zA-Z]+', rule_string)
# Helper function to parse expressions recursively
def parse_expression(tokens):
stack = []
while tokens:
token = tokens.pop(0)
if token == '(':
stack.append(parse_expression(tokens))
elif token == ')':
break
elif token in ('AND', 'OR'):
right = stack.pop()
left = stack.pop()
node = Node(node_type='operator', value=token, left=left, right=right)
stack.append(node)
elif re.match(r'\w+|>|<|=', token):
# Assume binary comparison, form operand
left = token
operator = tokens.pop(0)
right = tokens.pop(0)
stack.append(Node(node_type='operand', value=f"{left} {operator} {right}"))
return stack[0]
return parse_expression(tokens)
Step 3: Combining Rules
This function will combine multiple rules into one. We can join them using AND/OR and build the AST
accordingly. A heuristic (like most frequent operator) can be added later for optimization. python
def combine_rules(rule_nodes, operator='AND'): """Combines multiple rules into one AST using a given operator. :param rule_nodes: List of rule nodes (ASTs). :param operator: Logical operator to combine (default is 'AND'). :return: Combined AST root node. """
root = rule_nodes[0]
for rule in rule_nodes[1:]:
root = Node(node_type='operator', value=operator, left=root, right=rule)
return root
Step 4: Evaluating Rules
This function takes the AST and evaluates it based on the provided user data. python
def evaluate_rule(node, data): """Recursively evaluate an AST node against provided data. :param node: Root node of the AST. :param data: Dictionary of user data (e.g., {"age": 35, "department": "Sales"}). :return: Boolean indicating if the rule matches."""
if node.type == 'operand':
# Parse the condition (e.g., 'age > 30')
attribute, operator, value = node.value.split()
value = int(value) if value.isdigit() else value.strip("'")
if operator == '>':
return data[attribute] > value
elif operator == '<':
return data[attribute] < value
elif operator == '=':
return data[attribute] == value
elif node.type == 'operator':
if node.value == 'AND':
return evaluate_rule(node.left, data) and evaluate_rule(node.right, data)
elif node.value == 'OR':
return evaluate_rule(node.left, data) or evaluate_rule(node.right, data)
Step 5: Data Storage:
For storing rules and metadata, we can use a relational database like PostgreSQL. Here's an example
schema:
Relational Database Sample Schema (PostgreSQL):
Sql:
CREATE TABLE rules (
rule_id SERIAL PRIMARY KEY, rule_name VARCHAR(255), rule_ast JSONB -- Store the AST representation in JSON format
);
Document-Based Schema (MongoDB):
Json:
{ "rule_id": 1, "rule_name": "Sample Rule", "rule_ast": { "type": "operator", "value": "AND", "left": { "type": "operator", "value": "OR", "left": {"type": "operand", "value": "age > 30"}, "right": {"type": "operand", "value": "department = 'Sales'"}
},"right": { "type": "operator", "value": "OR", "left": {"type": "operand", "value": "salary > 50000"}, "right": {"type": "operand", "value": "experience > 5"}
}
}
}
Step 6: API Design
We can implement the API endpoints using a framework like Flask:
python
from flask import Flask, request, jsonify
app = Flask(__name__)
@app.route('/create_rule', methods=['POST'])
def api_create_rule():
rule_string = request.json['rule_string']
ast = create_rule(rule_string)
return jsonify(ast.__repr__())
@app.route('/combine_rules', methods=['POST'])
def api_combine_rules():
rules = [create_rule(r) for r in request.json['rules']]
combined_ast = combine_rules(rules)
return jsonify(combined_ast.__repr__())
@app.route('/evaluate_rule', methods=['POST'])
def api_evaluate_rule():
ast = create_rule(request.json['rule_string'])
user_data = request.json['data']
result = evaluate_rule(ast, user_data)
return jsonify({'result': result})
if __name__ == '__main__':
app.run(debug=True)
Step 7: Test Cases
I.Create individual rules:
python
rule1 = create_rule("((age > 30 AND department = 'Sales') OR (age < 25 AND department =
'Marketing')) AND (salary > 50000 OR experience > 5)")
print(rule1)
2.Combine rules:
python
combined_rule = combine_rules([rule1, rule2])
print(combined_rule)
3.Test evaluation:
python
data = {"age": 35, "department": "Sales", "salary": 60000, "experience": 3}
result = evaluate_rule(combined_rule, data)
print(result) # Should print True or False
4.Additional rule tests:
python
rule3 = create_rule("age > 40 AND department = 'HR'")
combined_rule = combine_rules([rule1, rule3], operator='OR')
print(combined_rule)
Step 8: Bonus Features
Error Handling: Wrap the parsing logic in try-except blocks to catch invalid rules. Rule Modification: Allow modifications to AST by finding and altering specific nodes in the tree. Catalog Validation: Pre-define allowed attributes in a catalog and validate rule inputs against it. Example for Test Cases and Bonus Features:
import re
class Node:
def __init__(self, node_type, value=None, left=None, right=None): """Represents a node in the AST. :param node_type: 'operator' for AND/OR or 'operand' for conditions. :param value: Value for operand nodes, None for operators. :param left: Left child node (if applicable). :param right: Right child node (if applicable). """
self.type = node_type # 'operator' or 'operand' self.value = value # e.g., age > 30
self.left = left # Left child node
self.right = right # Right child node
def __repr__(self):
if self.type == 'operand':
return f"Operand({self.value})" else:
return f"Operator({self.value})" class RuleEngine:
def __init__(self, allowed_attributes):
self.allowed_attributes = allowed_attributes
def create_rule(self, rule_string): """ Parse the rule string into an AST. :param rule_string: String representing the rule. :return: Root node of the AST or raises ValueError for invalid rules. """ # Tokenize the rule
tokens = re.findall(r'\(|\)|AND|OR|>|<|=|\'[^\']*\'|\d+|[a-zA-Z]+', rule_string)
# Validate tokens against allowed attributes
for token in tokens:
if token not in ('AND', 'OR', '(', ')', '>', '<', '=', *self.allowed_attributes):
raise ValueError(f"Invalid token in rule: {token}")
# Helper function to parse expressions recursively
def parse_expression(tokens):
stack = []
while tokens:
token = tokens.pop(0)
if token == '(':
stack.append(parse_expression(tokens))
elif token == ')':
break
elif token in ('AND', 'OR'):
right = stack.pop()
left = stack.pop()
node = Node(node_type='operator', value=token, left=left, right=right)
stack.append(node)
elif re.match(r'\w+|>|<|=', token):
# Assume binary comparison, form operand
left = token
operator = tokens.pop(0)
right = tokens.pop(0)
node = Node(node_type='operand', value=f"{left} {operator} {right}")
stack.append(node)
return stack[0]
return parse_expression(tokens)
def combine_rules(self, rule_nodes, operator='AND'): """ Combines multiple rules into one AST using a given operator. :param rule_nodes: List of rule nodes (ASTs). :param operator: Logical operator to combine (default is 'AND'). :return: Combined AST root node. """
root = rule_nodes[0]
for rule in rule_nodes[1:]:
root = Node(node_type='operator', value=operator, left=root, right=rule)
return root
def evaluate_rule(self, node, data): """ Recursively evaluate an AST node against provided data. :param node: Root node of the AST.
:param data: Dictionary of user data. :return: Boolean indicating if the rule matches. """
if node.type == 'operand':
# Parse the condition (e.g., 'age > 30')
attribute, operator, value = node.value.split()
value = int(value) if value.isdigit() else value.strip("'")
if operator == '>':
return data[attribute] > value
elif operator == '<':
return data[attribute] < value
elif operator == '=':
return data[attribute] == value
elif node.type == 'operator':
if node.value == 'AND':
return self.evaluate_rule(node.left, data) and self.evaluate_rule(node.right, data)
elif node.value == 'OR':
return self.evaluate_rule(node.left, data) or self.evaluate_rule(node.right, data)
def modify_rule(self, node, attribute, new_value): """ Modify an existing rule in the AST. :param node: The root node of the AST to modify. :param attribute: The attribute to change. :param new_value: The new value to set for the attribute. :return: Modified AST node. """
if node.type == 'operand':
# Check if this operand's attribute matches
if attribute in node.value:
operator, value = node.value.split()[1], node.value.split()[2]
node.value = f"{attribute} {operator} {new_value}" else:
# Recurse on children
if node.left:
self.modify_rule(node.left, attribute, new_value)
if node.right:
self.modify_rule(node.right, attribute, new_value)
return node
# Test Cases
if __name__ == "__main__":
allowed_attributes = ['age', 'department', 'salary', 'experience']
engine = RuleEngine(allowed_attributes)
# Test Case 1: Create individual rules
print("Test Case 1: Create Individual Rules")
try:
rule1 = engine.create_rule("((age > 30 AND department = 'Sales') OR (age < 25 AND department =
'Marketing')) AND (salary > 50000 OR experience > 5)")
print("Rule 1 AST:", rule1)
rule2 = engine.create_rule("((age > 30 AND department = 'Marketing')) AND (salary > 20000 OR
experience > 5)")
print("Rule 2 AST:", rule2)
except ValueError as e:
print("Error:", e)
# Test Case 2: Combine rules
print("\nTest Case 2: Combine Rules")
combined_rule = engine.combine_rules([rule1, rule2])
print("Combined Rule AST:", combined_rule)
# Test Case 3: Evaluate rules
print("\nTest Case 3: Evaluate Rules")
data = {"age": 35, "department": "Sales", "salary": 60000, "experience": 3}
result = engine.evaluate_rule(combined_rule, data)
print("Evaluation Result for data:", data, "=>", result)
# Test Case 4: Modify rules
print("\nTest Case 4: Modify Rule")
modified_rule = engine.modify_rule(rule1, 'age', '40')
print("Modified Rule 1 AST:", modified_rule)
# Test Case 5: Error Handling for Invalid Rules
print("\nTest Case 5: Error Handling for Invalid Rules")
try:
invalid_rule = engine.create_rule("((age > 30 AND dept = 'Sales') OR (age < 25 AND department =
'Marketing'))")
except ValueError as e:
print("Error:", e)
# Test Case 6: Evaluate with different data
print("\nTest Case 6: Evaluate with Different Data")
data2 = {"age": 22, "department": "Marketing", "salary": 40000, "experience": 6}
result2 = engine.evaluate_rule(combined_rule, data2)
print("Evaluation Result for data:", data2, "=>", result2)
# Test Case 7: Modify an existing rule to change the department
print("\nTest Case 7: Modify an Existing Rule")
modified_rule2 = engine.modify_rule(rule2, 'department', "'HR'")
print("Modified Rule 2 AST:", modified_rule2)
# Test Case 8: Evaluate modified rules
print("\nTest Case 8: Evaluate Modified Rules")
result3 = engine.evaluate_rule(modified_rule2, data)
print("Evaluation Result for modified rule 2 with data:", data, "=>", result3)
