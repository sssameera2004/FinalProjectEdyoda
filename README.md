class FoodItem:
    def __init__(self, name, price=0):
        self.name = name
        self.price = price

class Order:
    def __init__(self, name, items):
        self.name = name
        self.items = items

class Restaurant:
    def __init__(self, name, capacity):
        self.name = name
        self.capacity = int(capacity)
        self.menu = {}

    def add_item(self, item):
        self.menu[item.name] = item.price

    def update_item(self, item):
        self.menu[item.name] = item.price

    def check_item(self, item):
        return item.name in self.menu

    def check_capacity(self):
        return self.capacity > 0

    def can_place_order(self, order):
        if not self.check_capacity():
            return False
        for item in order.items:
            if not self.check_item(item):
                return False
        return True

    def place_order(self, order):
        self.capacity -= 1

class OrderManager:
    restaurants = {}
    commands = []

    def add_restaurant(self, restaurant):
        self.restaurants[restaurant.name] = restaurant

    def get_restaurant(self, restaurant_name):
        return self.restaurants[restaurant_name]

    def place_order(self, order):
        restaurant = self.select_restaurant(order)
        restaurant.place_order()

    def create_item(self, item_name, item_price):
        return FoodItem(item_name, item_price)

    def add_item_to_menu(self, restaurant_name, item):
        restaurant = self.get_restaurant(restaurant_name)
        restaurant.add_item(item)

    def select_restaurant(self, order):
        for restaurant in self.restaurants.values():
            if restaurant.can_place_order(order):
                return restaurant

    def add_command(self, command_str):
        command = [s.strip() for s in command_str.split(',')]
        self.commands.append(command)

    def run_command(self, command):
        print("command", command)
        def add_restaurant():
            restaurant = Restaurant(command[2], command[3])
            self.add_restaurant(restaurant)

        def add_item():
            item = self.create_item(command[3], command[4])
            self.add_item_to_menu(command[2], item)

        def place_order():
            items = [FoodItem(i) for i in command[3:]]
            order = Order(command[2], items)
            rest = self.select_restaurant(order)
            rest.place_order(order)
            print("Order placed at", rest.name)
            for item in items:
                print(item.name, rest.menu[item.name])


        def update_price():
            restaurant = self.get_restaurant(command[2])
            item = self.create_item(command[3], command[4])
            restaurant.update_item(item)
            print(restaurant.menu)

        def dispatch_order():
            print('Dispatch order :', command[2])

        def default():
            print('Command not found')

        switcher = {
            'add-restaurant': add_restaurant,
            'add-item': add_item,
            'update-price': update_price,
            'place-order': place_order,
            'dispatch-order': dispatch_order
        }

        switcher.get(command[1], default)()

    def run_commands(self):
        self.commands = sorted(self.commands, key=lambda x: int(x[0]))
        for command in self.commands:
            self.run_command(command)
        self.commands = []


# Driver code
if __name__ == '__main__':
    order_manager = OrderManager()

    with open('restaurants.csv') as f:
        for line in f:
            order_manager.add_command(line)

    order_manager.run_commands()
    print('Restraurant data loaded')

    with open('commands.csv') as f:
        for line in f:
            order_manager.add_command(line)

    order_manager.run_commands()
