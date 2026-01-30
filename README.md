from abc import ABC, abstractmethod
from datetime import datetime, timedelta

# Abstract Base Class for Parking Pass
class ParkingPass(ABC):
    def __init__(self, pass_id):
        self.pass_id = pass_id

    @abstractmethod
    def is_valid(self):
        pass

# Monthly Pass
class MonthlyPass(ParkingPass):
    def __init__(self, pass_id, expiry_date):
        super().__init__(pass_id)
        self.expiry_date = expiry_date

    def is_valid(self):
        return datetime.now().date() <= self.expiry_date

# Single Entry Pass
class SingleEntryPass(ParkingPass):
    def __init__(self, pass_id):
        super().__init__(pass_id)
        self.used = False

    def is_valid(self):
        return not self.used

    def use_pass(self):
        self.used = True

# Car Class
class Car:
    def __init__(self, car_id):
        self.car_id = car_id
        self.entry_time = None
        self.exit_time = None

    def enter(self):
        self.entry_time = datetime.now()
        print(f"Car {self.car_id} entered at {self.entry_time}")

    def exit(self):
        self.exit_time = datetime.now()
        print(f"Car {self.car_id} exited at {self.exit_time}")

    def parking_duration(self):
        if self.entry_time and self.exit_time:
            return (self.exit_time - self.entry_time).total_seconds() / 3600  # hours
        return 0

# Fee Calculator
class FeeCalculator:
    def calculate_fee(self, duration, pricing_type="regular"):
        rate = 5  # $5 per hour for cars
        if pricing_type == "peak":
            rate *= 1.5
        elif pricing_type == "weekend":
            rate *= 0.8
        return round(rate * duration, 2)

# Parking System
class ParkingSystem:
    def __init__(self, total_spaces=300):
        self.total_spaces = total_spaces
        self.available_spaces = total_spaces
        self.parked_cars = []

    def allocate_space(self, car):
        if self.available_spaces > 0:
            self.parked_cars.append(car)
            self.available_spaces -= 1
            car.enter()
            print(f"Space allocated. Available spaces: {self.available_spaces}")
        else:
            print("No space available!")

    def release_space(self, car):
        if car in self.parked_cars:
            car.exit()
            self.parked_cars.remove(car)
            self.available_spaces += 1
            print(f"Space released. Available spaces: {self.available_spaces}")
        else:
            print("Car not found in the system!")

# Sample Usage
if __name__ == "__main__":
    parking_system = ParkingSystem()
    fee_calculator = FeeCalculator()

    # Car 1: Regular
    car1 = Car("CAR001")
    parking_system.allocate_space(car1)
    car1.entry_time -= timedelta(hours=3)  # simulate 3 hours parking
    parking_system.release_space(car1)
    fee = fee_calculator.calculate_fee(car1.parking_duration(), pricing_type="peak")
    print(f"Parking Fee for {car1.car_id}: ${fee}")

    # Car 2: Monthly Pass
    car2 = Car("CAR002")
    monthly_pass = MonthlyPass("MP001", expiry_date=datetime.now().date() + timedelta(days=30))
    if monthly_pass.is_valid():
        parking_system.allocate_space(car2)
        car2.entry_time -= timedelta(hours=2)
        parking_system.release_space(car2)
        print(f"Parking Fee for {car2.car_id} with monthly pass: $0")

    # Car 3: Single Entry Pass
    car3 = Car("CAR003")
    single_pass = SingleEntryPass("SEP001")
    if single_pass.is_valid():
        parking_system.allocate_space(car3)
        car3.entry_time -= timedelta(hours=1.5)
        parking_system.release_space(car3)
        fee = fee_calculator.calculate_fee(car3.parking_duration())
        single_pass.use_pass()
        print(f"Parking Fee for {car3.car_id} with single entry pass: ${fee}")


