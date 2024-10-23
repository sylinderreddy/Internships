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



python
import requests
import time
import sqlite3
import datetime
import matplotlib.pyplot as plt
class WeatherMonitor:
def __init__(self, api_key, cities, db_name='weather_data.db', interval=300):
self.api_key = api_key
self.cities = cities
self.db_name = db_name
self.interval = interval # Interval in seconds
self.temperature_threshold = None
self.alert_triggered = False
# Setup database
self.conn = sqlite3.connect(self.db_name)
self.create_table()
def create_table(self): """Create a table to store daily weather summaries.""" with self.conn:
self.conn.execute(''' CREATE TABLE IF NOT EXISTS daily_summary (
date TEXT PRIMARY KEY, avg_temp REAL, max_temp REAL, min_temp REAL, dominant_condition TEXT
)
''')
def fetch_weather_data(self): """Fetch weather data from the OpenWeatherMap API.""" weather_data = {}
for city in self.cities:
url =
f"http://api.openweathermap.org/data/2.5/weather?q={city},IN&appid={self.api_key}&units=metric"
response = requests.get(url)
if response.status_code == 200:
data = response.json()
weather_data[city] = {
'main': data['weather'][0]['main'],
'temp': data['main']['temp'],
'feels_like': data['main']['feels_like'],
'dt': data['dt']
}
else:
print(f"Failed to retrieve data for {city}. Status code: {response.status_code}")
return weather_data
def process_weather_data(self, weather_data): """Process the weather data and store daily summaries."""
today = datetime.date.today().isoformat()
if today not in self.get_stored_dates():
daily_summary = {
'avg_temp': 0,
'max_temp': float('-inf'),
'min_temp': float('inf'),
'conditions': {}
}
for city, data in weather_data.items():
temp = data['temp']
condition = data['main']
# Update daily summary
daily_summary['avg_temp'] += temp
daily_summary['max_temp'] = max(daily_summary['max_temp'], temp)
daily_summary['min_temp'] = min(daily_summary['min_temp'], temp)
# Count occurrences of each condition
if condition in daily_summary['conditions']:
daily_summary['conditions'][condition] += 1
else:
daily_summary['conditions'][condition] = 1
# Finalize average temperature
city_count = len(self.cities)
daily_summary['avg_temp'] /= city_count
# Determine dominant weather condition
dominant_condition = max(daily_summary['conditions'], key=daily_summary['conditions'].get)
# Save to database
self.save_daily_summary(today, daily_summary['avg_temp'], daily_summary['max_temp'], daily_summary['min_temp'], dominant_condition)
def save_daily_summary(self, date, avg_temp, max_temp, min_temp, dominant_condition):
"""Save the daily weather summary to the database.""" with self.conn:
self.conn.execute(''' INSERT INTO daily_summary (date, avg_temp, max_temp, min_temp, dominant_condition)
VALUES (?, ?, ?, ?, ?)''', (date, avg_temp, max_temp, min_temp, dominant_condition))
def get_stored_dates(self): """Retrieve stored dates from the database.""" with self.conn:
return [row[0] for row in self.conn.execute('SELECT date FROM daily_summary')]
def set_temperature_threshold(self, threshold): """Set a temperature threshold for alerts."""
self.temperature_threshold = threshold
def check_alerts(self, weather_data): """Check if any weather data breaches the alert threshold."""
for city, data in weather_data.items():
temp = data['temp']
if self.temperature_threshold and temp > self.temperature_threshold:
if not self.alert_triggered:
print(f"Alert! {city}: Temperature exceeded {self.temperature_threshold}°C. Current temp:
{temp}°C.")
self.alert_triggered = True
else:
self.alert_triggered = False
def visualize_data(self): """Visualize daily weather summaries using matplotlib.""" dates = []
avg_temps = []
max_temps = []
min_temps = []
with self.conn:
cursor = self.conn.execute('SELECT date, avg_temp, max_temp, min_temp FROM
daily_summary')
for row in cursor:
dates.append(row[0])
avg_temps.append(row[1])
max_temps.append(row[2])
min_temps.append(row[3])
plt.figure(figsize=(10, 5))
plt.plot(dates, avg_temps, label='Average Temperature', marker='o')
plt.plot(dates, max_temps, label='Maximum Temperature', marker='o')
plt.plot(dates, min_temps, label='Minimum Temperature', marker='o')
plt.title('Daily Weather Summary')
plt.xlabel('Date')
plt.ylabel('Temperature (°C)')
plt.xticks(rotation=45)
plt.legend()
plt.tight_layout()
plt.show()
def run(self): """Run the weather monitoring system."""
while True:
weather_data = self.fetch_weather_data()
self.process_weather_data(weather_data)
self.check_alerts(weather_data)
time.sleep(self.interval)
# Test the system
if __name__ == "__main__":
API_KEY = "your_api_key_here" # Replace with your OpenWeatherMap API key
CITIES = ['Delhi', 'Mumbai', 'Chennai', 'Bangalore', 'Kolkata', 'Hyderabad']
weather_monitor = WeatherMonitor(API_KEY, CITIES)
weather_monitor.set_temperature_threshold(35) # Set threshold for alerts
try:
weather_monitor.run()
except KeyboardInterrupt:
print("Stopping the weather monitor.")
weather_monitor.conn.close()
