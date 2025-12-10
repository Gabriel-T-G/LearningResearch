
Table of Contents

- Introduction (#introduction)
- Core Concepts (#core-concepts)
- Why Use Dependency Injection? (#why-use-dependency-injection)
- Types of Dependency Injection (#types-of-dependency-injection)
- Implementing DI Without Frameworks (#implementing-di-without-frameworks)
- Python Abstractions for DI (#python-abstractions-for-di)
- DI Frameworks and Libraries (#di-frameworks-and-libraries)
- Framework-Specific DI (#framework-specific-di)
- Testing with Dependency Injection (#testing-with-dependency-injection)
- Best Practices and Anti-Patterns (#best-practices-and-anti-patterns)
- Real-World Examples (#real-world-examples)
- When to Use (and Not Use) DI (#when-to-use-and-not-use-di)
- Additional Resources (#additional-resources)
-------------------------

Introduction

Dependency Injection (DI) is a design pattern in software development that manages how components in your application get their dependencies (other objects they need to work). Instead of a class creating its own dependencies internally, dependencies are "injected" from outside, making the code more maintainable, flexible, and easier to test.
DI is part of a larger software design principle known as Inversion of Control (IoC), where the control of dependency management shifts from the components themselves to an external handler.
> **Key Insight**: While DI is less frequently used in Python than in languages like Java or C#, it offers powerful design benefits for creating maintainable and testable code.

Sources:
- Snyk - The Ultimate Guide to Dependency Injection in Python (https://snyk.io/blog/dependency-injection-python/)
- Toptal - Python Dependency Injection Tutorial (https://www.toptal.com/python/python-dependency-injection-design-pattern)
-------------------------

Core Concepts

Inversion of Control (IoC)

IoC is a principle that reshapes our approach to designing maintainable and scalable software. By shifting the responsibility for managing dependencies from components to an external container or framework, we achieve:
- Loose coupling between components
- Better separation of concerns
- Improved testability

# WITHOUT IoC - Class controls its own dependencies
class OrderService:
    def __init__(self):
        # The class creates its own dependency - tight coupling!
        self.payment_gateway = PayPalGateway()  # Hardcoded dependency
    
    def process_order(self, order):
        self.payment_gateway.charge(order.total)

# WITH IoC - Dependencies are provided from outside
class OrderService:
    def __init__(self, payment_gateway):
        # Dependency is injected - loose coupling!
        self.payment_gateway = payment_gateway  # Any payment gateway can be used
    
    def process_order(self, order):
        self.payment_gateway.charge(order.total)


Dependency Inversion Principle (DIP)

The Dependency Inversion Principle is the "D" in SOLID principles. It states:
- High-level modules should not depend on low-level modules. Both should depend on abstractions.
- Abstractions should not depend on details. Details should depend on abstractions.

from abc import ABC, abstractmethod

# ABSTRACTION - Both high-level and low-level modules depend on this
class NotificationService(ABC):
    @abstractmethod
    def send(self, message: str, recipient: str) -> None:
        pass

# LOW-LEVEL MODULE - Implements the abstraction
class EmailNotificationService(NotificationService):
    def send(self, message: str, recipient: str) -> None:
        # Email sending logic
        print(f"Sending email to {recipient}: {message}")

class SMSNotificationService(NotificationService):
    def send(self, message: str, recipient: str) -> None:
        # SMS sending logic
        print(f"Sending SMS to {recipient}: {message}")

# HIGH-LEVEL MODULE - Depends on abstraction, not implementation
class UserController:
    def __init__(self, notification_service: NotificationService):
        # Depends on the abstract NotificationService, not concrete implementations
        self.notification_service = notification_service
    
    def notify_user(self, user_email: str, message: str):
        self.notification_service.send(message, user_email)

# Usage - We can easily swap implementations
email_service = EmailNotificationService()
sms_service = SMSNotificationService()

controller_with_email = UserController(email_service)
controller_with_sms = UserController(sms_service)


Sources:
- Dev.to - Learning SOLID principles through Python - Part 5: Dependency Inversion Principle (https://dev.to/alexmercedcoder/learning-solid-principles-through-python-part-5-dependency-inversion-principle-2n8i)
- Medium - SOLID Principles in Python: A Comprehensive Guide (https://medium.com/@mudasirhaji/solid-principles-in-python-a-comprehensive-guide-e5a99b423df3)
-------------------------

Why Use Dependency Injection?

Benefits

| Benefit | Description |
|---------|-------------|
| **Loose Coupling** | Components don't need to know how to create their dependencies |
| **Testability** | Easy to substitute mock objects during testing |
| **Maintainability** | Changes to dependencies don't require changes to dependent code |
| **Flexibility** | Easy to swap implementations without modifying client code |
| **Reusability** | Components can be reused with different dependencies |
| **Single Responsibility** | Classes focus on their core functionality, not dependency creation |



Without DI vs With DI


# ❌ WITHOUT DEPENDENCY INJECTION
class UserRepository:
    def __init__(self):
        # Tight coupling - hard to test, hard to change database
        self.database = PostgreSQLDatabase("localhost", "users_db")
    
    def get_user(self, user_id):
        return self.database.query(f"SELECT * FROM users WHERE id = {user_id}")

# Problems:
# 1. Can't easily test without a real database
# 2. Can't switch to a different database without modifying this class
# 3. Connection parameters are hardcoded



# ✅ WITH DEPENDENCY INJECTION
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def query(self, sql: str):
        pass

class UserRepository:
    def __init__(self, database: Database):
        # Loose coupling - database is injected
        self.database = database
    
    def get_user(self, user_id):
        return self.database.query(f"SELECT * FROM users WHERE id = {user_id}")

# Benefits:
# 1. Easy to test with a mock database
# 2. Can switch databases by injecting different implementation
# 3. Configuration is external to the class


Sources:
- DataCamp - Dependency Injection in Python: Everything You Need to Know (https://www.datacamp.com/tutorial/dependency-injection-python)
- Stackademic - Understanding Dependency Injection in Python (https://blog.stackademic.com/understanding-dependency-injection-in-python-5b3eef0a1b73)
-------------------------

Types of Dependency Injection

There are three main types of dependency injection in Python:
1. Constructor Injection (Most Common)

Dependencies are provided through the class constructor (__init__). This is the most common and recommended approach.

from abc import ABC, abstractmethod

# Step 1: Define the interface (abstraction)
class MessageService(ABC):
    @abstractmethod
    def send_message(self, recipient: str, message: str) -> bool:
        """Send a message to a recipient"""
        pass

# Step 2: Create concrete implementations
class EmailService(MessageService):
    def send_message(self, recipient: str, message: str) -> bool:
        # In reality, this would send an actual email
        print(f"Email sent to {recipient}: {message}")
        return True

class SlackService(MessageService):
    def send_message(self, recipient: str, message: str) -> bool:
        # In reality, this would post to Slack
        print(f"Slack message to {recipient}: {message}")
        return True

# Step 3: Create the class that uses constructor injection
class NotificationManager:
    def __init__(self, message_service: MessageService):
        """
        Constructor injection - the dependency is passed in during object creation.
        The class doesn't know or care which specific implementation it receives.
        """
        self._message_service = message_service
    
    def notify(self, recipient: str, message: str) -> bool:
        # Use the injected dependency
        return self._message_service.send_message(recipient, message)

# Step 4: Usage - inject the dependency when creating the object
if __name__ == "__main__":
    # Create different implementations
    email_service = EmailService()
    slack_service = SlackService()
    
    # Inject email service
    email_notifier = NotificationManager(email_service)
    email_notifier.notify("user@example.com", "Hello via email!")
    
    # Inject slack service - same class, different behavior
    slack_notifier = NotificationManager(slack_service)
    slack_notifier.notify("@username", "Hello via Slack!")


When to use: For required dependencies that the class cannot function without.
2. Setter Injection (Property Injection)

Dependencies are provided through setter methods after object creation.

class DataProcessor:
    def __init__(self):
        # Initialize with no logger - it's optional
        self._logger = None
    
    @property
    def logger(self):
        """Getter for the logger property"""
        return self._logger
    
    @logger.setter
    def logger(self, logger):
        """
        Setter injection - allows setting the dependency after object creation.
        This is useful for optional dependencies.
        """
        self._logger = logger
    
    def process(self, data):
        # Only log if a logger was injected
        if self._logger:
            self._logger.info(f"Processing data: {data}")
        
        # Process the data
        result = data.upper()  # Simple example transformation
        
        if self._logger:
            self._logger.info(f"Processing complete: {result}")
        
        return result

# Usage
import logging

# Create the processor without any logger
processor = DataProcessor()

# The processor works without a logger
result = processor.process("hello")  # Works fine, just no logging

# Later, inject a logger using the setter
logging.basicConfig(level=logging.INFO)
processor.logger = logging.getLogger(__name__)

# Now it will log
result = processor.process("hello with logging")


When to use: For optional dependencies that aren't required for basic functionality.
3. Method Injection (Interface Injection)

Dependencies are provided as method parameters when the method is called.

from abc import ABC, abstractmethod

class Formatter(ABC):
    @abstractmethod
    def format(self, data: dict) -> str:
        pass

class JSONFormatter(Formatter):
    def format(self, data: dict) -> str:
        import json
        return json.dumps(data, indent=2)

class XMLFormatter(Formatter):
    def format(self, data: dict) -> str:
        # Simplified XML formatting
        xml_parts = ['<?xml version="1.0"?>']
        xml_parts.append('<data>')
        for key, value in data.items():
            xml_parts.append(f'  <{key}>{value}</{key}>')
        xml_parts.append('</data>')
        return '\n'.join(xml_parts)

class ReportGenerator:
    def __init__(self, data: dict):
        """The data is a constructor dependency"""
        self.data = data
    
    def generate_report(self, formatter: Formatter) -> str:
        """
        Method injection - the formatter is passed directly to the method.
        This allows different formatters for different method calls.
        """
        return formatter.format(self.data)

# Usage
report_data = {
    "name": "Sales Report",
    "total": 10000,
    "period": "Q4 2024"
}

generator = ReportGenerator(report_data)

# Use JSON formatter for this call
json_formatter = JSONFormatter()
json_report = generator.generate_report(json_formatter)
print("JSON Report:")
print(json_report)

# Use XML formatter for another call
xml_formatter = XMLFormatter()
xml_report = generator.generate_report(xml_formatter)
print("\nXML Report:")
print(xml_report)


When to use: When you need different dependencies for different method calls, or when the dependency varies based on context.
Sources:
- Towards Dev - Dependency Injection in Python (https://towardsdev.com/dependency-injection-in-python-2456dd37dfdb)
- Snyk - The Ultimate Guide to Dependency Injection in Python (https://snyk.io/blog/dependency-injection-python/)
- DataCamp - Dependency Injection in Python (https://www.datacamp.com/tutorial/dependency-injection-python)
-------------------------

Implementing DI Without Frameworks

For small to medium projects, you can implement dependency injection without any framework. This is often called "Pure DI" or "Poor Man's DI".
Simple Manual DI


from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List

# ============== INTERFACES (ABSTRACTIONS) ==============

class UserRepository(ABC):
    """Interface for user data access"""
    @abstractmethod
    def get_by_id(self, user_id: int) -> dict:
        pass
    
    @abstractmethod
    def save(self, user: dict) -> None:
        pass

class EmailService(ABC):
    """Interface for sending emails"""
    @abstractmethod
    def send_email(self, to: str, subject: str, body: str) -> bool:
        pass

# ============== CONCRETE IMPLEMENTATIONS ==============

class PostgreSQLUserRepository(UserRepository):
    """PostgreSQL implementation of UserRepository"""
    
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        # In reality, you'd establish a database connection here
        print(f"Connected to PostgreSQL: {connection_string}")
    
    def get_by_id(self, user_id: int) -> dict:
        # Simulated database query
        return {"id": user_id, "name": "John Doe", "email": "john@example.com"}
    
    def save(self, user: dict) -> None:
        print(f"Saving user to PostgreSQL: {user}")

class InMemoryUserRepository(UserRepository):
    """In-memory implementation for testing"""
    
    def __init__(self):
        self._users = {}
    
    def get_by_id(self, user_id: int) -> dict:
        return self._users.get(user_id, {})
    
    def save(self, user: dict) -> None:
        self._users[user["id"]] = user

class SMTPEmailService(EmailService):
    """SMTP implementation of EmailService"""
    
    def __init__(self, smtp_host: str, smtp_port: int):
        self.smtp_host = smtp_host
        self.smtp_port = smtp_port
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        print(f"Sending email via SMTP to {to}: {subject}")
        return True

class MockEmailService(EmailService):
    """Mock implementation for testing"""
    
    def __init__(self):
        self.sent_emails: List[dict] = []
    
    def send_email(self, to: str, subject: str, body: str) -> bool:
        self.sent_emails.append({"to": to, "subject": subject, "body": body})
        return True

# ============== SERVICE LAYER (HIGH-LEVEL MODULE) ==============

class UserService:
    """
    Service class that depends on abstractions (UserRepository, EmailService).
    Dependencies are injected through the constructor.
    """
    
    def __init__(
        self,
        user_repository: UserRepository,
        email_service: EmailService
    ):
        # Store injected dependencies
        self._user_repository = user_repository
        self._email_service = email_service
    
    def register_user(self, user_data: dict) -> dict:
        """Register a new user and send welcome email"""
        # Save user using the injected repository
        self._user_repository.save(user_data)
        
        # Send welcome email using the injected email service
        self._email_service.send_email(
            to=user_data["email"],
            subject="Welcome!",
            body=f"Hello {user_data['name']}, welcome to our platform!"
        )
        
        return user_data
    
    def get_user(self, user_id: int) -> dict:
        """Get user by ID"""
        return self._user_repository.get_by_id(user_id)

# ============== COMPOSITION ROOT ==============
# This is where all dependencies are wired together

def create_production_user_service() -> UserService:
    """
    Factory function that creates UserService with production dependencies.
    This is your 'Composition Root' - the single place where the object graph is built.
    """
    # Create concrete implementations
    user_repo = PostgreSQLUserRepository("postgresql://localhost/mydb")
    email_service = SMTPEmailService("smtp.example.com", 587)
    
    # Inject dependencies and return the service
    return UserService(
        user_repository=user_repo,
        email_service=email_service
    )

def create_test_user_service() -> UserService:
    """
    Factory function that creates UserService with test dependencies.
    Same service class, but with mock implementations.
    """
    return UserService(
        user_repository=InMemoryUserRepository(),
        email_service=MockEmailService()
    )

# ============== USAGE ==============

if __name__ == "__main__":
    # Production usage
    print("=== Production Environment ===")
    service = create_production_user_service()
    service.register_user({"id": 1, "name": "Alice", "email": "alice@example.com"})
    
    # Test usage
    print("\n=== Test Environment ===")
    test_service = create_test_user_service()
    test_service.register_user({"id": 2, "name": "Bob", "email": "bob@test.com"})


Using a Simple DI Container (DIY)


from typing import TypeVar, Type, Dict, Callable, Any

T = TypeVar('T')

class SimpleDIContainer:
    """
    A simple dependency injection container.
    Stores factory functions for creating instances of registered types.
    """
    
    def __init__(self):
        # Dictionary to store type -> factory function mappings
        self._services: Dict[Type, Callable[[], Any]] = {}
        # Cache for singleton instances
        self._singletons: Dict[Type, Any] = {}
        # Set of types registered as singletons
        self._singleton_types: set = set()
    
    def register(self, interface: Type[T], factory: Callable[[], T]) -> None:
        """
        Register a factory function for creating instances of a type.
        
        Args:
            interface: The type/interface to register
            factory: A callable that returns an instance of that type
        """
        self._services[interface] = factory
    
    def register_singleton(self, interface: Type[T], factory: Callable[[], T]) -> None:
        """
        Register a singleton - only one instance will be created and reused.
        
        Args:
            interface: The type/interface to register
            factory: A callable that returns an instance of that type
        """
        self._services[interface] = factory
        self._singleton_types.add(interface)
    
    def register_instance(self, interface: Type[T], instance: T) -> None:
        """
        Register an existing instance as a singleton.
        
        Args:
            interface: The type/interface to register
            instance: The pre-created instance to use
        """
        self._singletons[interface] = instance
        self._singleton_types.add(interface)
        self._services[interface] = lambda: instance
    
    def resolve(self, interface: Type[T]) -> T:
        """
        Resolve (get an instance of) a registered type.
        
        Args:
            interface: The type to resolve
            
        Returns:
            An instance of the requested type
            
        Raises:
            KeyError: If the type is not registered
        """
        if interface not in self._services:
            raise KeyError(f"Service {interface.__name__} is not registered")
        
        # Check if this is a singleton and already created
        if interface in self._singleton_types:
            if interface not in self._singletons:
                # Create singleton instance for the first time
                self._singletons[interface] = self._services[interface]()
            return self._singletons[interface]
        
        # Create new instance for transient services
        return self._services[interface]()

# ============== USAGE WITH THE CONTAINER ==============

# Create container instance
container = SimpleDIContainer()

# Register services
container.register(UserRepository, lambda: InMemoryUserRepository())
container.register_singleton(EmailService, lambda: SMTPEmailService("smtp.example.com", 587))

# Register UserService with its dependencies resolved from the container
container.register(
    UserService,
    lambda: UserService(
        user_repository=container.resolve(UserRepository),
        email_service=container.resolve(EmailService)
    )
)

# Resolve and use the service
user_service = container.resolve(UserService)
user = user_service.get_user(1)


Sources:
- YourDevOps - Python Dependency Injection: A Comprehensive Guide (https://www.yourdevops.org/python-dependency-injection)
- Reddit - Do you do dependency injection without a framework? (https://www.reddit.com/r/Python/comments/179sbcv/do_you_do_dependency_injection_without_a_framework/)
-------------------------

Python Abstractions for DI

Python offers two main ways to define interfaces/abstractions for dependency injection:
1. Abstract Base Classes (ABC)

The traditional approach using Python's abc module.

from abc import ABC, abstractmethod
from typing import List

class PaymentProcessor(ABC):
    """
    Abstract Base Class defining the interface for payment processing.
    Any class that inherits from this MUST implement all abstract methods.
    """
    
    @abstractmethod
    def process_payment(self, amount: float) -> bool:
        """Process a payment of the given amount"""
        pass
    
    @abstractmethod
    def refund(self, transaction_id: str, amount: float) -> bool:
        """Refund a previous transaction"""
        pass
    
    @property
    @abstractmethod
    def provider_name(self) -> str:
        """Return the name of the payment provider"""
        pass

class StripeProcessor(PaymentProcessor):
    """Concrete implementation for Stripe payments"""
    
    def __init__(self, api_key: str):
        self._api_key = api_key
    
    def process_payment(self, amount: float) -> bool:
        print(f"Processing ${amount} via Stripe")
        return True
    
    def refund(self, transaction_id: str, amount: float) -> bool:
        print(f"Refunding ${amount} for transaction {transaction_id} via Stripe")
        return True
    
    @property
    def provider_name(self) -> str:
        return "Stripe"

class PayPalProcessor(PaymentProcessor):
    """Concrete implementation for PayPal payments"""
    
    def __init__(self, client_id: str, client_secret: str):
        self._client_id = client_id
        self._client_secret = client_secret
    
    def process_payment(self, amount: float) -> bool:
        print(f"Processing ${amount} via PayPal")
        return True
    
    def refund(self, transaction_id: str, amount: float) -> bool:
        print(f"Refunding ${amount} for transaction {transaction_id} via PayPal")
        return True
    
    @property
    def provider_name(self) -> str:
        return "PayPal"

# This would raise TypeError - can't instantiate abstract class
# processor = PaymentProcessor()  # TypeError!

# These work fine
stripe = StripeProcessor("sk_test_xxx")
paypal = PayPalProcessor("client_id", "client_secret")


2. Protocols (Structural Typing) - Python 3.8+

Protocols provide duck typing with static type checking support. Unlike ABCs, classes don't need to explicitly inherit from a Protocol.

from typing import Protocol, runtime_checkable

@runtime_checkable
class Serializer(Protocol):
    """
    Protocol defining what a serializer should look like.
    Any class with these methods is considered a Serializer - no inheritance needed!
    This is called "structural typing" or "static duck typing".
    """
    
    def serialize(self, data: dict) -> str:
        """Convert data to string format"""
        ...  # The ... (ellipsis) indicates this is a protocol method signature
    
    def deserialize(self, data: str) -> dict:
        """Convert string back to data"""
        ...

# These classes don't inherit from Serializer, but they're still compatible!

class JSONSerializer:
    """JSON serializer - implicitly implements Serializer protocol"""
    
    def serialize(self, data: dict) -> str:
        import json
        return json.dumps(data)
    
    def deserialize(self, data: str) -> dict:
        import json
        return json.loads(data)

class YAMLSerializer:
    """YAML serializer - also implicitly implements Serializer protocol"""
    
    def serialize(self, data: dict) -> str:
        import yaml
        return yaml.dump(data)
    
    def deserialize(self, data: str) -> dict:
        import yaml
        return yaml.safe_load(data)

class DataStore:
    """Uses the Serializer protocol for type hints"""
    
    def __init__(self, serializer: Serializer):
        # Type checker knows this must have serialize/deserialize methods
        self._serializer = serializer
    
    def save(self, key: str, data: dict) -> None:
        serialized = self._serializer.serialize(data)
        print(f"Saving {key}: {serialized}")
    
    def load(self, key: str) -> dict:
        # In reality, this would load from storage
        raw_data = '{"loaded": true}'
        return self._serializer.deserialize(raw_data)

# Both work without inheriting from Serializer!
json_store = DataStore(JSONSerializer())
yaml_store = DataStore(YAMLSerializer())

# Runtime check (only works with @runtime_checkable)
print(isinstance(JSONSerializer(), Serializer))  # True


ABC vs Protocol: When to Use Which?

| Aspect | ABC | Protocol |
|--------|-----|----------|
| **Inheritance** | Required | Not required |
| **Coupling** | Explicit | Loose |
| **Python Version** | All versions | 3.8+ |
| **Best For** | Framework APIs, when you control implementations | Library code, when you don't control implementations |
| **Runtime isinstance()** | Always works | Requires `@runtime_checkable` |



Sources:
- Dev.to - Why Use Abstractions in Dependency Injection (https://dev.to/dstarner/why-use-abstractions-in-dependency-injection-exploring-the-benefits-in-python-25mp)
- Medium - Duck typing, Abstractions and Protocols (https://medium.com/@shashwat2704/duck-typing-abstractions-and-protocols-7e8f51d9b21d)
- DbVis - Python Protocols: How They Work, Examples & Best Practices (https://www.dbvis.com/thetable/python-protocols/)
-------------------------

DI Frameworks and Libraries

1. dependency-injector (Most Popular)

The most comprehensive DI framework for Python with support for various provider types.
Installation:

pip install dependency-injector


Key Features:
- Factory, Singleton, Callable, and other providers
- Configuration management
- Async support
- Wiring (automatic injection)

from dependency_injector import containers, providers
from dependency_injector.wiring import Provide, inject

# ============== SERVICE DEFINITIONS ==============

class DatabaseClient:
    """Simulates a database client"""
    
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
        print(f"DatabaseClient connected to {host}:{port}")
    
    def query(self, sql: str):
        return f"Results from {self.host}: {sql}"

class CacheClient:
    """Simulates a cache client"""
    
    def __init__(self, host: str):
        self.host = host
        print(f"CacheClient connected to {host}")
    
    def get(self, key: str):
        return f"Cached value for {key}"

class UserService:
    """Service that uses database and cache"""
    
    def __init__(self, db: DatabaseClient, cache: CacheClient):
        self.db = db
        self.cache = cache
    
    def get_user(self, user_id: int):
        # First check cache
        cached = self.cache.get(f"user:{user_id}")
        # Then fall back to database
        return self.db.query(f"SELECT * FROM users WHERE id = {user_id}")

# ============== CONTAINER DEFINITION ==============

class Container(containers.DeclarativeContainer):
    """
    Dependency Injection Container.
    This is the central configuration for all your dependencies.
    """
    
    # Configuration provider - loads settings from various sources
    config = providers.Configuration()
    
    # Singleton provider - creates one instance and reuses it
    # Great for database connections, cache clients, etc.
    database_client = providers.Singleton(
        DatabaseClient,
        host=config.database.host,
        port=config.database.port,
    )
    
    # Another singleton for cache
    cache_client = providers.Singleton(
        CacheClient,
        host=config.cache.host,
    )
    
    # Factory provider - creates new instance on each call
    # Dependencies are automatically injected!
    user_service = providers.Factory(
        UserService,
        db=database_client,  # Automatically resolves the singleton
        cache=cache_client,  # Automatically resolves the singleton
    )

# ============== USAGE ==============

if __name__ == "__main__":
    # Create container instance
    container = Container()
    
    # Load configuration (can be from YAML, INI, env vars, etc.)
    container.config.from_dict({
        "database": {
            "host": "localhost",
            "port": 5432
        },
        "cache": {
            "host": "redis://localhost"
        }
    })
    
    # Resolve dependencies
    # Factory creates new instance each time
    service1 = container.user_service()
    service2 = container.user_service()
    
    print(f"service1 is service2: {service1 is service2}")  # False - different instances
    print(f"Same DB client: {service1.db is service2.db}")  # True - singleton
    
    # Use the service
    result = service1.get_user(42)
    print(result)


Wiring Feature (Automatic Injection):

from dependency_injector.wiring import Provide, inject

# Mark function for injection
@inject
def process_request(
    user_service: UserService = Provide[Container.user_service]
):
    """
    The @inject decorator and Provide marker enable automatic injection.
    When this function is called, dependency_injector will automatically
    resolve and inject the user_service dependency.
    """
    return user_service.get_user(1)

# Wire the container to modules that use injection
container = Container()
container.config.from_dict({...})
container.wire(modules=[__name__])  # Wire to current module

# Now call the function - dependencies are auto-injected!
result = process_request()  # No need to pass user_service


Sources:
- Dependency Injector Documentation (https://python-dependency-injector.ets-labs.org/)
- GitHub - python-dependency-injector (https://github.com/ets-labs/python-dependency-injector)
2. injector Library

Inspired by Google's Guice framework for Java.
Installation:

pip install injector



from injector import Injector, inject, singleton, Module, provider

# ============== SERVICE DEFINITIONS ==============

class Logger:
    def log(self, message: str):
        print(f"LOG: {message}")

class Database:
    @inject
    def __init__(self, logger: Logger):
        """The @inject decorator tells injector to resolve Logger automatically"""
        self.logger = logger
        self.logger.log("Database initialized")
    
    def query(self, sql: str):
        self.logger.log(f"Executing: {sql}")
        return {"result": "data"}

class UserRepository:
    @inject
    def __init__(self, database: Database, logger: Logger):
        """Multiple dependencies are resolved automatically"""
        self.database = database
        self.logger = logger
    
    def find_user(self, user_id: int):
        self.logger.log(f"Finding user {user_id}")
        return self.database.query(f"SELECT * FROM users WHERE id = {user_id}")

# ============== MODULE (CONFIGURATION) ==============

class AppModule(Module):
    """
    Module defines how dependencies should be created.
    Similar to Spring's Java configuration or .NET's ConfigureServices.
    """
    
    @singleton
    @provider
    def provide_logger(self) -> Logger:
        """
        @singleton ensures only one Logger instance is created.
        @provider marks this as a factory method for Logger.
        """
        return Logger()
    
    @provider
    def provide_database(self, logger: Logger) -> Database:
        """
        Dependencies in provider methods are automatically resolved.
        This is NOT a singleton, so new instance each time.
        """
        return Database(logger)

# ============== USAGE ==============

if __name__ == "__main__":
    # Create injector with our module
    injector = Injector([AppModule()])
    
    # Resolve the full dependency graph
    # Injector automatically creates: Logger -> Database -> UserRepository
    user_repo = injector.get(UserRepository)
    
    # Use the service
    user = user_repo.find_user(42)
    print(user)
    
    # Get another UserRepository - new instance, but same Logger (singleton)
    user_repo2 = injector.get(UserRepository)
    print(f"Same repo: {user_repo is user_repo2}")  # False
    print(f"Same logger: {user_repo.logger is user_repo2.logger}")  # True


Sources:
- injector PyPI (https://pypi.org/project/injector/)
- injector Documentation (https://injector.readthedocs.io/en/latest/)
- Snyk - Getting Started With DI Using the injector Library (https://snyk.io/blog/getting-started-with-di-in-python-using-the-injector-library/)
3. punq (Lightweight)

A simple, lightweight DI container with no dependencies.
Installation:

pip install punq



import punq

# ============== SERVICE DEFINITIONS ==============

class IMessageService:
    """Interface for message services (using duck typing)"""
    def send(self, message: str) -> None:
        raise NotImplementedError

class EmailMessageService:
    """Email implementation"""
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class SMSMessageService:
    """SMS implementation"""
    def send(self, message: str) -> None:
        print(f"SMS: {message}")

class NotificationService:
    """High-level service that depends on IMessageService"""
    def __init__(self, message_service: IMessageService):
        self.message_service = message_service
    
    def notify(self, message: str) -> None:
        self.message_service.send(f"Notification: {message}")

# ============== CONTAINER SETUP ==============

# Create container
container = punq.Container()

# Register implementations
# Syntax: container.register(Interface, Implementation)
container.register(IMessageService, EmailMessageService)

# Register service with auto-wired dependencies
container.register(NotificationService)

# ============== USAGE ==============

# Resolve and use
notification_service = container.resolve(NotificationService)
notification_service.notify("Hello from punq!")

# Switch implementations easily
container2 = punq.Container()
container2.register(IMessageService, SMSMessageService)
container2.register(NotificationService)

sms_notification = container2.resolve(NotificationService)
sms_notification.notify("Hello via SMS!")


Sources:
- Medium - Lightweight Python DI Container: punq (https://medium.com/@lifanov.a.v/lightweight-python-di-container-punq-35d27e3d1581)
4. kink (Simple and Modern)

Focuses on simplicity and developer experience.
Installation:

pip install kink



from kink import di, inject

# ============== SERVICE DEFINITIONS ==============

class Settings:
    def __init__(self):
        self.debug = True
        self.api_key = "secret-key"

class APIClient:
    def __init__(self, settings: Settings):
        self.settings = settings
    
    def call(self, endpoint: str):
        return f"Calling {endpoint} with key {self.settings.api_key}"

# ============== REGISTRATION ==============

# Register services in the container
di[Settings] = Settings()  # Register singleton instance
di[APIClient] = lambda di: APIClient(di[Settings])  # Factory with dependency

# ============== USAGE ==============

# Using @inject decorator
@inject
def make_api_call(client: APIClient):
    """
    @inject automatically resolves APIClient from the container
    """
    return client.call("/users")

# Call the function - APIClient is injected automatically
result = make_api_call()
print(result)

# Or resolve manually
client = di[APIClient]
print(client.call("/products"))


Sources:
- kink PyPI (https://pypi.org/project/kink/)
- GitHub - kodemore/kink (https://github.com/kodemore/kink)
5. lagom

Designed to provide "just enough" help with dependency injection.
Installation:

pip install lagom



from lagom import Container

# ============== SERVICE DEFINITIONS ==============

class Database:
    def __init__(self, connection_string: str = "sqlite:///default.db"):
        self.connection_string = connection_string

class UserRepository:
    def __init__(self, db: Database):
        self.db = db
    
    def get_user(self, user_id: int):
        return f"User {user_id} from {self.db.connection_string}"

# ============== CONTAINER SETUP ==============

container = Container()

# Define how to construct Database with specific config
container[Database] = Database("postgresql://localhost/myapp")

# UserRepository will be auto-wired (Database resolved automatically)
# No explicit registration needed!

# ============== USAGE ==============

# Resolve with automatic dependency injection
user_repo = container[UserRepository]
print(user_repo.get_user(1))

# The Database is the same instance (singleton by default)
user_repo2 = container[UserRepository]
print(user_repo.db is user_repo2.db)  # True


Sources:
- lagom Documentation (https://lagom-di.readthedocs.io/en/latest/)
Framework Comparison

| Framework | Complexity | Features | Best For |
|-----------|-----------|----------|----------|
| **dependency-injector** | High | Comprehensive, async support, wiring | Large applications |
| **injector** | Medium | Guice-style, modules, scopes | Java developers, medium apps |
| **punq** | Low | Lightweight, no deps | Small to medium apps |
| **kink** | Low | Simple, decorator-based | Quick setup, microservices |
| **lagom** | Low | Auto-wiring | Minimal configuration needs |



-------------------------

Framework-Specific DI

FastAPI Dependency Injection

FastAPI has built-in, powerful dependency injection using the Depends function.

from fastapi import FastAPI, Depends, HTTPException
from typing import Annotated

app = FastAPI()

# ============== DEPENDENCIES ==============

class Database:
    """Simulated database connection"""
    def __init__(self):
        self.connected = True
    
    def query(self, sql: str):
        return {"data": "results"}

# Dependency function - called for each request
def get_database() -> Database:
    """
    This is a dependency function.
    FastAPI will call this to get the Database instance.
    """
    db = Database()
    try:
        yield db  # Provide the dependency
    finally:
        # Cleanup code runs after the request
        db.connected = False
        print("Database connection closed")

# More complex dependency with sub-dependencies
class UserRepository:
    def __init__(self, db: Database):
        self.db = db
    
    def get_user(self, user_id: int):
        return self.db.query(f"SELECT * FROM users WHERE id = {user_id}")

def get_user_repository(
    db: Annotated[Database, Depends(get_database)]
) -> UserRepository:
    """
    This dependency depends on get_database!
    FastAPI handles the chain automatically.
    """
    return UserRepository(db)

# ============== ROUTES ==============

@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    # Dependency injection via Depends()
    user_repo: Annotated[UserRepository, Depends(get_user_repository)]
):
    """
    The user_repo parameter is automatically injected by FastAPI.
    The dependency chain is: get_database -> get_user_repository -> this function
    """
    user = user_repo.get_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# ============== REUSABLE DEPENDENCIES ==============

# Create reusable dependency type alias
UserRepoDep = Annotated[UserRepository, Depends(get_user_repository)]

@app.get("/users/{user_id}/profile")
async def get_profile(user_id: int, user_repo: UserRepoDep):
    """Using the reusable dependency type"""
    return user_repo.get_user(user_id)

# ============== CLASS-BASED DEPENDENCIES ==============

class Pagination:
    """Class that acts as a callable dependency"""
    
    def __init__(self, skip: int = 0, limit: int = 10):
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def list_items(
    pagination: Annotated[Pagination, Depends(Pagination)]
):
    """
    Pagination class is used as a dependency.
    Query parameters (skip, limit) are automatically extracted.
    """
    return {
        "skip": pagination.skip,
        "limit": pagination.limit,
        "items": []
    }


Sources:
- FastAPI - Dependencies (https://fastapi.tiangolo.com/tutorial/dependencies/)
- Better Stack - Dependency injection in FastAPI (advanced) (https://betterstack.com/community/guides/scaling-python/fastapi-dependency-injection/)
Flask with Dependency Injection

Flask doesn't have built-in DI, but you can use the dependency-injector library or implement your own.

from flask import Flask, jsonify
from dependency_injector import containers, providers
from dependency_injector.wiring import Provide, inject

# ============== SERVICES ==============

class UserRepository:
    def get_all(self):
        return [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
    
    def get_by_id(self, user_id: int):
        users = self.get_all()
        return next((u for u in users if u["id"] == user_id), None)

class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
    
    def list_users(self):
        return self.repository.get_all()
    
    def get_user(self, user_id: int):
        return self.repository.get_by_id(user_id)

# ============== CONTAINER ==============

class Container(containers.DeclarativeContainer):
    """DI Container for Flask app"""
    
    wiring_config = containers.WiringConfiguration(
        modules=[__name__]  # Wire to this module
    )
    
    # Register services
    user_repository = providers.Singleton(UserRepository)
    user_service = providers.Factory(
        UserService,
        repository=user_repository
    )

# ============== FLASK APP ==============

def create_app() -> Flask:
    """Application factory with DI setup"""
    
    # Create and configure container
    container = Container()
    
    # Create Flask app
    app = Flask(__name__)
    app.container = container
    
    # ============== ROUTES WITH INJECTION ==============
    
    @app.route("/users")
    @inject
    def list_users(
        user_service: UserService = Provide[Container.user_service]
    ):
        """
        The @inject decorator enables automatic injection.
        user_service is provided by the container.
        """
        return jsonify(user_service.list_users())
    
    @app.route("/users/<int:user_id>")
    @inject
    def get_user(
        user_id: int,
        user_service: UserService = Provide[Container.user_service]
    ):
        user = user_service.get_user(user_id)
        if user is None:
            return jsonify({"error": "Not found"}), 404
        return jsonify(user)
    
    return app

# ============== RUN ==============

if __name__ == "__main__":
    app = create_app()
    app.run(debug=True)


Sources:
- Dependency Injector - Flask Tutorial (https://python-dependency-injector.ets-labs.org/tutorials/index.html)
- Snyk - Building testable Flask applications with DI (https://snyk.io/blog/dependency-injection-python/)
Django Dependency Injection

Django doesn't have built-in DI, but you can implement patterns manually or use libraries.

# services.py
from abc import ABC, abstractmethod

class EmailBackend(ABC):
    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> bool:
        pass

class SMTPEmailBackend(EmailBackend):
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
    
    def send(self, to: str, subject: str, body: str) -> bool:
        print(f"Sending via SMTP {self.host}:{self.port}")
        return True

class SESEmailBackend(EmailBackend):
    def __init__(self, region: str):
        self.region = region
    
    def send(self, to: str, subject: str, body: str) -> bool:
        print(f"Sending via AWS SES in {self.region}")
        return True

class NotificationService:
    def __init__(self, email_backend: EmailBackend):
        self.email_backend = email_backend
    
    def notify_user(self, user_email: str, message: str):
        return self.email_backend.send(
            to=user_email,
            subject="Notification",
            body=message
        )

# container.py - Simple service locator for Django
from django.conf import settings

class ServiceContainer:
    """Simple service container for Django"""
    
    _instances = {}
    
    @classmethod
    def get_email_backend(cls) -> EmailBackend:
        if 'email_backend' not in cls._instances:
            # Configure based on Django settings
            if settings.EMAIL_BACKEND_TYPE == 'ses':
                cls._instances['email_backend'] = SESEmailBackend(
                    region=settings.AWS_REGION
                )
            else:
                cls._instances['email_backend'] = SMTPEmailBackend(
                    host=settings.EMAIL_HOST,
                    port=settings.EMAIL_PORT
                )
        return cls._instances['email_backend']
    
    @classmethod
    def get_notification_service(cls) -> NotificationService:
        if 'notification_service' not in cls._instances:
            cls._instances['notification_service'] = NotificationService(
                email_backend=cls.get_email_backend()
            )
        return cls._instances['notification_service']

# views.py
from django.http import JsonResponse
from .container import ServiceContainer

def send_notification(request):
    """View that uses injected service"""
    notification_service = ServiceContainer.get_notification_service()
    
    success = notification_service.notify_user(
        user_email=request.POST.get('email'),
        message=request.POST.get('message')
    )
    
    return JsonResponse({"success": success})


Sources:
- Stack Overflow - How to do Dependency Injection in Django? (https://stackoverflow.com/questions/58455541/how-to-do-dependency-injection-in-django)
- Toptal - Python Dependency Injection - Django (https://www.toptal.com/python/python-dependency-injection-design-pattern)
-------------------------

Testing with Dependency Injection

One of the main benefits of DI is improved testability. Here's how to leverage DI for testing:
Using Mock Objects


import unittest
from unittest.mock import Mock, MagicMock, patch
from abc import ABC, abstractmethod

# ============== PRODUCTION CODE ==============

class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, card_token: str) -> dict:
        pass

class StripeGateway(PaymentGateway):
    def charge(self, amount: float, card_token: str) -> dict:
        # Real implementation would call Stripe API
        pass

class OrderService:
    def __init__(self, payment_gateway: PaymentGateway):
        self.payment_gateway = payment_gateway
    
    def process_order(self, order_id: str, amount: float, card_token: str) -> dict:
        # Process the payment
        result = self.payment_gateway.charge(amount, card_token)
        
        if result.get("success"):
            return {"order_id": order_id, "status": "completed", "transaction_id": result["transaction_id"]}
        else:
            return {"order_id": order_id, "status": "failed", "error": result.get("error")}

# ============== TESTS ==============

class TestOrderService(unittest.TestCase):
    """Unit tests for OrderService using mock dependencies"""
    
    def test_process_order_success(self):
        """Test successful order processing with mock payment gateway"""
        
        # Create a mock payment gateway
        mock_gateway = Mock(spec=PaymentGateway)
        
        # Configure the mock to return a successful response
        mock_gateway.charge.return_value = {
            "success": True,
            "transaction_id": "txn_123"
        }
        
        # Inject the mock into OrderService
        order_service = OrderService(payment_gateway=mock_gateway)
        
        # Call the method under test
        result = order_service.process_order(
            order_id="order_456",
            amount=99.99,
            card_token="tok_test"
        )
        
        # Assertions
        self.assertEqual(result["status"], "completed")
        self.assertEqual(result["transaction_id"], "txn_123")
        
        # Verify the mock was called correctly
        mock_gateway.charge.assert_called_once_with(99.99, "tok_test")
    
    def test_process_order_failure(self):
        """Test failed order processing"""
        
        # Create mock that simulates a failed payment
        mock_gateway = Mock(spec=PaymentGateway)
        mock_gateway.charge.return_value = {
            "success": False,
            "error": "Card declined"
        }
        
        order_service = OrderService(payment_gateway=mock_gateway)
        
        result = order_service.process_order(
            order_id="order_789",
            amount=50.00,
            card_token="tok_declined"
        )
        
        # Verify failure handling
        self.assertEqual(result["status"], "failed")
        self.assertEqual(result["error"], "Card declined")

if __name__ == "__main__":
    unittest.main()


Pytest Fixtures as Dependency Injection

Pytest fixtures are a form of dependency injection for tests.

import pytest
from abc import ABC, abstractmethod

# ============== PRODUCTION CODE ==============

class UserRepository(ABC):
    @abstractmethod
    def get_by_id(self, user_id: int) -> dict:
        pass
    
    @abstractmethod
    def save(self, user: dict) -> dict:
        pass

class InMemoryUserRepository(UserRepository):
    """In-memory implementation for testing"""
    
    def __init__(self):
        self._users = {}
        self._next_id = 1
    
    def get_by_id(self, user_id: int) -> dict:
        return self._users.get(user_id)
    
    def save(self, user: dict) -> dict:
        if 'id' not in user:
            user['id'] = self._next_id
            self._next_id += 1
        self._users[user['id']] = user
        return user

class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
    
    def create_user(self, name: str, email: str) -> dict:
        user = {"name": name, "email": email}
        return self.repository.save(user)
    
    def get_user(self, user_id: int) -> dict:
        return self.repository.get_by_id(user_id)

# ============== PYTEST FIXTURES (DI FOR TESTS) ==============

@pytest.fixture
def user_repository():
    """
    Fixture that provides an in-memory repository.
    This is dependency injection for tests!
    """
    return InMemoryUserRepository()

@pytest.fixture
def user_service(user_repository):
    """
    Fixture that provides UserService with injected repository.
    Note how user_repository is automatically injected by pytest.
    """
    return UserService(repository=user_repository)

@pytest.fixture
def sample_user(user_service):
    """
    Fixture that creates a sample user for tests.
    Demonstrates fixture chaining/composition.
    """
    return user_service.create_user(name="Test User", email="test@example.com")

# ============== TESTS ==============

def test_create_user(user_service):
    """
    Test receives user_service fixture automatically.
    No need to create dependencies manually!
    """
    user = user_service.create_user(name="Alice", email="alice@example.com")
    
    assert user["name"] == "Alice"
    assert user["email"] == "alice@example.com"
    assert "id" in user

def test_get_user(user_service, sample_user):
    """
    Test receives both fixtures - demonstrates DI composition.
    sample_user fixture uses user_service fixture internally.
    """
    retrieved = user_service.get_user(sample_user["id"])
    
    assert retrieved == sample_user

def test_user_not_found(user_service):
    """Test handling of non-existent user"""
    result = user_service.get_user(999)
    
    assert result is None

# ============== FIXTURE SCOPES ==============

@pytest.fixture(scope="module")
def database_connection():
    """
    Module-scoped fixture - created once per test module.
    Similar to singleton scope in DI containers.
    """
    print("Creating database connection")
    connection = {"connected": True}
    yield connection
    print("Closing database connection")
    connection["connected"] = False

@pytest.fixture(scope="session")
def app_config():
    """
    Session-scoped fixture - created once for entire test session.
    Useful for expensive setup operations.
    """
    return {
        "debug": True,
        "database_url": "sqlite:///:memory:"
    }


Testing with DI Container Overrides


from dependency_injector import containers, providers
from dependency_injector.wiring import Provide, inject

# ============== SERVICES ==============

class ExternalAPIClient:
    def fetch_data(self):
        # In reality, this calls an external API
        return {"source": "real_api"}

class DataProcessor:
    def __init__(self, api_client: ExternalAPIClient):
        self.api_client = api_client
    
    def process(self):
        data = self.api_client.fetch_data()
        return {"processed": True, **data}

# ============== CONTAINER ==============

class Container(containers.DeclarativeContainer):
    api_client = providers.Singleton(ExternalAPIClient)
    data_processor = providers.Factory(
        DataProcessor,
        api_client=api_client
    )

# ============== TESTS WITH OVERRIDES ==============

import pytest

class MockAPIClient:
    """Mock implementation for testing"""
    def fetch_data(self):
        return {"source": "mock_api", "test_data": True}

@pytest.fixture
def container():
    """Fixture that provides configured container"""
    container = Container()
    return container

@pytest.fixture
def container_with_mock(container):
    """
    Fixture that overrides real dependencies with mocks.
    This is the recommended way to test with DI containers.
    """
    # Override the api_client with mock
    with container.api_client.override(MockAPIClient()):
        yield container

def test_data_processor_with_mock(container_with_mock):
    """Test using overridden container"""
    processor = container_with_mock.data_processor()
    result = processor.process()
    
    # Verify mock was used
    assert result["source"] == "mock_api"
    assert result["test_data"] is True
    assert result["processed"] is True

def test_data_processor_real(container):
    """Test without override uses real implementation"""
    processor = container.data_processor()
    result = processor.process()
    
    assert result["source"] == "real_api"


Sources:
- pytest Documentation - How to Use Fixtures (https://docs.pytest.org/en/stable/how-to/fixtures.html)
- Snyk - Pytest fixtures as dependency injection (https://snyk.io/blog/dependency-injection-python/)
- Reddit - Testing and Mocking with Dependency Injection in Python (https://www.reddit.com/r/Python/comments/b8npb2/testing_and_mocking_with_dependency_injection_in/)
-------------------------

Best Practices and Anti-Patterns

Best Practices

1. Use Constructor Injection for Required Dependencies


# ✅ GOOD: Constructor injection for required dependencies
class OrderProcessor:
    def __init__(
        self,
        payment_service: PaymentService,
        inventory_service: InventoryService,
        notification_service: NotificationService
    ):
        """All required dependencies are visible in the constructor"""
        self._payment_service = payment_service
        self._inventory_service = inventory_service
        self._notification_service = notification_service


2. Use Setter Injection for Optional Dependencies


# ✅ GOOD: Setter injection for optional dependencies
class ReportGenerator:
    def __init__(self, data_source: DataSource):
        self._data_source = data_source
        self._logger = None  # Optional
        self._cache = None   # Optional
    
    @property
    def logger(self):
        return self._logger
    
    @logger.setter
    def logger(self, logger: Logger):
        """Optional logger can be set after construction"""
        self._logger = logger


3. Depend on Abstractions, Not Implementations


# ❌ BAD: Depending on concrete implementation
class UserService:
    def __init__(self, repo: PostgreSQLUserRepository):  # Concrete type!
        self.repo = repo

# ✅ GOOD: Depending on abstraction
class UserService:
    def __init__(self, repo: UserRepository):  # Abstract type!
        self.repo = repo


4. Keep Constructors Simple


# ❌ BAD: Complex logic in constructor
class DataService:
    def __init__(self, config_path: str):
        # Loading files, making connections in constructor = BAD
        self.config = json.load(open(config_path))
        self.db = psycopg2.connect(self.config["database_url"])
        self.cache = redis.Redis(self.config["cache_url"])

# ✅ GOOD: Constructor only assigns dependencies
class DataService:
    def __init__(
        self,
        config: Config,
        db: DatabaseConnection,
        cache: CacheClient
    ):
        # Just assign - no complex logic
        self.config = config
        self.db = db
        self.cache = cache


5. Use a Single Composition Root


# ✅ GOOD: Single place where all dependencies are wired
# composition_root.py

def create_application() -> Application:
    """
    Composition Root - the single place where the object graph is built.
    This is the only place that knows about concrete implementations.
    """
    # Create all dependencies
    config = load_config()
    database = create_database_connection(config)
    cache = create_cache_client(config)
    
    # Wire up the dependency graph
    user_repo = PostgreSQLUserRepository(database)
    notification_service = EmailNotificationService(config.smtp)
    user_service = UserService(user_repo, notification_service)
    
    return Application(user_service)


Anti-Patterns to Avoid

1. Service Locator Anti-Pattern


# ❌ BAD: Service Locator pattern - hides dependencies
class UserService:
    def create_user(self, name: str):
        # Dependencies are hidden - discovered at runtime
        repo = ServiceLocator.get(UserRepository)
        email = ServiceLocator.get(EmailService)
        
        user = repo.create(name)
        email.send_welcome(user)
        return user

# Problems:
# 1. Dependencies are hidden from the constructor
# 2. Hard to test - need to configure ServiceLocator
# 3. Hard to understand what the class needs


2. Captive Dependencies


# ❌ BAD: Singleton holding reference to shorter-lived dependency
class Container:
    # Singleton that holds a Factory-scoped dependency
    user_service = providers.Singleton(
        UserService,
        # UserRepository should be per-request, not singleton!
        repository=providers.Factory(UserRepository)  # Will only be called once!
    )

# ✅ GOOD: Match lifetimes appropriately
class Container:
    user_repository = providers.Factory(UserRepository)  # Per-request
    user_service = providers.Factory(  # Also per-request
        UserService,
        repository=user_repository
    )


3. Constructor Over-Injection


# ❌ BAD: Too many dependencies - class does too much
class MegaService:
    def __init__(
        self,
        user_repo: UserRepository,
        order_repo: OrderRepository,
        product_repo: ProductRepository,
        payment_service: PaymentService,
        email_service: EmailService,
        sms_service: SMSService,
        logger: Logger,
        cache: Cache,
        metrics: MetricsClient
    ):
        # Too many dependencies = class has too many responsibilities
        pass

# ✅ GOOD: Split into smaller, focused services
class OrderService:
    def __init__(
        self,
        order_repo: OrderRepository,
        payment_service: PaymentService
    ):
        # Fewer dependencies = single responsibility
        pass


4. Ambient Context Anti-Pattern


# ❌ BAD: Using global/static context
class UserService:
    def get_current_user(self):
        # Accessing global state - hidden dependency
        return CurrentUserContext.user

# ✅ GOOD: Pass context explicitly
class UserService:
    def get_user_data(self, user: User):
        # User is passed explicitly
        return self.repo.get_data_for(user)


Sources:
- DataCamp - Dependency Injection in Python Best Practices (https://www.datacamp.com/tutorial/dependency-injection-python)
- Solution Perspective - Dependency Injection Anti-Patterns (https://solutionperspective.blog/2020/08/04/dependency-injection-anti-patterns/)
- Toptal - Python Dependency Injection Tutorial (https://www.toptal.com/python/python-dependency-injection-design-pattern)
-------------------------

Real-World Examples

Example 1: E-Commerce Order Processing


from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime
from enum import Enum

# ============== DOMAIN MODELS ==============

class OrderStatus(Enum):
    PENDING = "pending"
    PAID = "paid"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

@dataclass
class OrderItem:
    product_id: str
    quantity: int
    price: float

@dataclass
class Order:
    id: str
    customer_id: str
    items: List[OrderItem]
    status: OrderStatus
    created_at: datetime
    total: float = 0.0
    
    def __post_init__(self):
        self.total = sum(item.price * item.quantity for item in self.items)

# ============== INTERFACES (PORTS) ==============

class OrderRepository(ABC):
    """Interface for order persistence"""
    
    @abstractmethod
    def save(self, order: Order) -> Order:
        pass
    
    @abstractmethod
    def get_by_id(self, order_id: str) -> Optional[Order]:
        pass
    
    @abstractmethod
    def get_by_customer(self, customer_id: str) -> List[Order]:
        pass

class PaymentProcessor(ABC):
    """Interface for payment processing"""
    
    @abstractmethod
    def process_payment(self, order: Order, payment_method: dict) -> dict:
        pass
    
    @abstractmethod
    def refund(self, transaction_id: str, amount: float) -> dict:
        pass

class InventoryService(ABC):
    """Interface for inventory management"""
    
    @abstractmethod
    def check_availability(self, product_id: str, quantity: int) -> bool:
        pass
    
    @abstractmethod
    def reserve(self, product_id: str, quantity: int) -> bool:
        pass
    
    @abstractmethod
    def release(self, product_id: str, quantity: int) -> bool:
        pass

class NotificationService(ABC):
    """Interface for sending notifications"""
    
    @abstractmethod
    def notify_order_placed(self, order: Order) -> None:
        pass
    
    @abstractmethod
    def notify_order_shipped(self, order: Order, tracking_number: str) -> None:
        pass

# ============== CONCRETE IMPLEMENTATIONS ==============

class PostgreSQLOrderRepository(OrderRepository):
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        # In reality: establish database connection
    
    def save(self, order: Order) -> Order:
        print(f"Saving order {order.id} to PostgreSQL")
        return order
    
    def get_by_id(self, order_id: str) -> Optional[Order]:
        print(f"Fetching order {order_id} from PostgreSQL")
        return None  # Simplified
    
    def get_by_customer(self, customer_id: str) -> List[Order]:
        return []

class StripePaymentProcessor(PaymentProcessor):
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def process_payment(self, order: Order, payment_method: dict) -> dict:
        print(f"Processing ${order.total} via Stripe")
        return {"success": True, "transaction_id": f"stripe_{order.id}"}
    
    def refund(self, transaction_id: str, amount: float) -> dict:
        return {"success": True, "refund_id": f"refund_{transaction_id}"}

class WarehouseInventoryService(InventoryService):
    def __init__(self, warehouse_api_url: str):
        self.api_url = warehouse_api_url
    
    def check_availability(self, product_id: str, quantity: int) -> bool:
        return True  # Simplified
    
    def reserve(self, product_id: str, quantity: int) -> bool:
        print(f"Reserving {quantity} of {product_id}")
        return True
    
    def release(self, product_id: str, quantity: int) -> bool:
        return True

class EmailNotificationService(NotificationService):
    def __init__(self, smtp_config: dict):
        self.smtp_config = smtp_config
    
    def notify_order_placed(self, order: Order) -> None:
        print(f"Sending order confirmation email for {order.id}")
    
    def notify_order_shipped(self, order: Order, tracking_number: str) -> None:
        print(f"Sending shipping notification for {order.id}")

# ============== APPLICATION SERVICE (ORCHESTRATOR) ==============

class OrderService:
    """
    Application service that orchestrates order processing.
    All dependencies are injected through the constructor.
    """
    
    def __init__(
        self,
        order_repository: OrderRepository,
        payment_processor: PaymentProcessor,
        inventory_service: InventoryService,
        notification_service: NotificationService
    ):
        self._order_repo = order_repository
        self._payment = payment_processor
        self._inventory = inventory_service
        self._notification = notification_service
    
    def place_order(
        self,
        customer_id: str,
        items: List[dict],
        payment_method: dict
    ) -> Order:
        """
        Place a new order - orchestrates multiple services.
        """
        # 1. Create order items
        order_items = [
            OrderItem(
                product_id=item["product_id"],
                quantity=item["quantity"],
                price=item["price"]
            )
            for item in items
        ]
        
        # 2. Check inventory for all items
        for item in order_items:
            if not self._inventory.check_availability(item.product_id, item.quantity):
                raise ValueError(f"Product {item.product_id} not available")
        
        # 3. Create order
        order = Order(
            id=f"ORD-{datetime.now().timestamp()}",
            customer_id=customer_id,
            items=order_items,
            status=OrderStatus.PENDING,
            created_at=datetime.now()
        )
        
        # 4. Reserve inventory
        for item in order_items:
            self._inventory.reserve(item.product_id, item.quantity)
        
        try:
            # 5. Process payment
            payment_result = self._payment.process_payment(order, payment_method)
            
            if not payment_result.get("success"):
                # Release inventory on payment failure
                for item in order_items:
                    self._inventory.release(item.product_id, item.quantity)
                raise ValueError("Payment failed")
            
            # 6. Update order status
            order.status = OrderStatus.PAID
            
            # 7. Save order
            order = self._order_repo.save(order)
            
            # 8. Send notification
            self._notification.notify_order_placed(order)
            
            return order
            
        except Exception as e:
            # Rollback inventory reservation on any error
            for item in order_items:
                self._inventory.release(item.product_id, item.quantity)
            raise

# ============== COMPOSITION ROOT ==============

def create_order_service(config: dict) -> OrderService:
    """
    Factory function that creates OrderService with all dependencies.
    This is the composition root for the order processing domain.
    """
    return OrderService(
        order_repository=PostgreSQLOrderRepository(config["database_url"]),
        payment_processor=StripePaymentProcessor(config["stripe_api_key"]),
        inventory_service=WarehouseInventoryService(config["warehouse_api_url"]),
        notification_service=EmailNotificationService(config["smtp"])
    )

# ============== USAGE ==============

if __name__ == "__main__":
    config = {
        "database_url": "postgresql://localhost/orders",
        "stripe_api_key": "sk_test_xxx",
        "warehouse_api_url": "https://warehouse.example.com",
        "smtp": {"host": "smtp.example.com", "port": 587}
    }
    
    order_service = create_order_service(config)
    
    order = order_service.place_order(
        customer_id="cust_123",
        items=[
            {"product_id": "prod_1", "quantity": 2, "price": 29.99},
            {"product_id": "prod_2", "quantity": 1, "price": 49.99}
        ],
        payment_method={"type": "card", "token": "tok_visa"}
    )
    
    print(f"Order placed: {order.id}, Total: ${order.total}")


Example 2: Async Application with Resource Management


import asyncio
from abc import ABC, abstractmethod
from typing import AsyncIterator
from contextlib import asynccontextmanager

# ============== ASYNC INTERFACES ==============

class AsyncDatabasePool(ABC):
    @abstractmethod
    async def connect(self) -> None:
        pass
    
    @abstractmethod
    async def disconnect(self) -> None:
        pass
    
    @abstractmethod
    async def execute(self, query: str) -> list:
        pass

class AsyncCacheClient(ABC):
    @abstractmethod
    async def connect(self) -> None:
        pass
    
    @abstractmethod
    async def disconnect(self) -> None:
        pass
    
    @abstractmethod
    async def get(self, key: str) -> str:
        pass
    
    @abstractmethod
    async def set(self, key: str, value: str, ttl: int = 300) -> None:
        pass

# ============== ASYNC IMPLEMENTATIONS ==============

class AsyncPGPool(AsyncDatabasePool):
    def __init__(self, connection_string: str, min_size: int = 5, max_size: int = 20):
        self.connection_string = connection_string
        self.min_size = min_size
        self.max_size = max_size
        self._pool = None
    
    async def connect(self) -> None:
        print(f"Connecting to PostgreSQL pool...")
        # In reality: self._pool = await asyncpg.create_pool(...)
        await asyncio.sleep(0.1)  # Simulate connection
        print("PostgreSQL pool connected")
    
    async def disconnect(self) -> None:
        print("Closing PostgreSQL pool...")
        # In reality: await self._pool.close()
        await asyncio.sleep(0.1)
        print("PostgreSQL pool closed")
    
    async def execute(self, query: str) -> list:
        # In reality: return await self._pool.fetch(query)
        return [{"result": "data"}]

class AsyncRedisClient(AsyncCacheClient):
    def __init__(self, url: str):
        self.url = url
        self._client = None
    
    async def connect(self) -> None:
        print(f"Connecting to Redis...")
        await asyncio.sleep(0.1)
        print("Redis connected")
    
    async def disconnect(self) -> None:
        print("Closing Redis connection...")
        await asyncio.sleep(0.1)
        print("Redis closed")
    
    async def get(self, key: str) -> str:
        return f"cached:{key}"
    
    async def set(self, key: str, value: str, ttl: int = 300) -> None:
        print(f"Caching {key}")

# ============== APPLICATION SERVICE ==============

class UserDataService:
    """Service that uses async database and cache"""
    
    def __init__(self, db: AsyncDatabasePool, cache: AsyncCacheClient):
        self._db = db
        self._cache = cache
    
    async def get_user(self, user_id: str) -> dict:
        # Try cache first
        cached = await self._cache.get(f"user:{user_id}")
        if cached:
            return {"source": "cache", "data": cached}
        
        # Fall back to database
        result = await self._db.execute(f"SELECT * FROM users WHERE id = '{user_id}'")
        
        # Cache the result
        if result:
            await self._cache.set(f"user:{user_id}", str(result[0]))
        
        return {"source": "database", "data": result}

# ============== LIFECYCLE MANAGEMENT ==============

class ApplicationContainer:
    """Container that manages async resource lifecycle"""
    
    def __init__(self, config: dict):
        self.config = config
        self._db: AsyncDatabasePool = None
        self._cache: AsyncCacheClient = None
        self._user_service: UserDataService = None
    
    async def startup(self) -> None:
        """Initialize all resources"""
        print("Starting application...")
        
        # Create instances
        self._db = AsyncPGPool(
            connection_string=self.config["database_url"],
            min_size=self.config.get("db_pool_min", 5),
            max_size=self.config.get("db_pool_max", 20)
        )
        self._cache = AsyncRedisClient(url=self.config["redis_url"])
        
        # Connect resources
        await self._db.connect()
        await self._cache.connect()
        
        # Create services with injected dependencies
        self._user_service = UserDataService(db=self._db, cache=self._cache)
        
        print("Application started")
    
    async def shutdown(self) -> None:
        """Cleanup all resources"""
        print("Shutting down application...")
        await self._cache.disconnect()
        await self._db.disconnect()
        print("Application shut down")
    
    @property
    def user_service(self) -> UserDataService:
        return self._user_service

@asynccontextmanager
async def create_application(config: dict) -> AsyncIterator[ApplicationContainer]:
    """
    Async context manager for application lifecycle.
    Ensures proper startup and shutdown.
    """
    container = ApplicationContainer(config)
    try:
        await container.startup()
        yield container
    finally:
        await container.shutdown()

# ============== USAGE ==============

async def main():
    config = {
        "database_url": "postgresql://localhost/myapp",
        "redis_url": "redis://localhost:6379",
        "db_pool_min": 5,
        "db_pool_max": 20
    }
    
    # Use context manager for proper lifecycle management
    async with create_application(config) as app:
        # Application is ready to use
        user_service = app.user_service
        
        # Use the service
        result = await user_service.get_user("user_123")
        print(f"Got user: {result}")
        
        # Make more calls
        result2 = await user_service.get_user("user_456")
        print(f"Got user: {result2}")
    
    # Resources are automatically cleaned up when exiting the context

if __name__ == "__main__":
    asyncio.run(main())


Sources:
- YourDevOps - Python Dependency Injection: Real-World Examples (https://www.yourdevops.org/python-dependency-injection)
- Dependency Injector - Async injections (https://python-dependency-injector.ets-labs.org/providers/async.html)
- Dependency Injector - Resource providers (https://python-dependency-injector.ets-labs.org/providers/resource.html)
-------------------------

When to Use (and Not Use) DI

When to Use Dependency Injection

| Scenario | Why DI Helps |
|----------|--------------|
| **Large applications** | Manages complexity, clear dependency graph |
| **Team projects** | Makes dependencies explicit and documented |
| **Code that needs testing** | Easy to substitute mock objects |
| **Flexible configurations** | Swap implementations without code changes |
| **External dependencies** | Databases, APIs, file systems can be easily mocked |
| **Long-term projects** | Easier to maintain and refactor |



When NOT to Use Dependency Injection

| Scenario | Why DI Might Be Overkill |
|----------|--------------------------|
| **Simple scripts** | Adds unnecessary complexity |
| **Small utilities** | Over-engineering for simple tools |
| **Prototypes** | Slows down rapid development |
| **No external dependencies** | Pure functions don't need DI |
| **Performance-critical hot paths** | Minimal overhead, but still overhead |



Decision Framework


Should I use Dependency Injection?

1. Is this a simple script or utility?
   YES → Skip DI, keep it simple
   NO  → Continue...

2. Does the code have external dependencies (DB, APIs, files)?
   YES → Use DI for easier testing
   NO  → Continue...

3. Will multiple people work on this code?
   YES → Use DI for explicit dependencies
   NO  → Continue...

4. Will this code need to be tested?
   YES → Use DI for mockability
   NO  → Continue...

5. Will implementations need to be swapped?
   YES → Use DI for flexibility
   NO  → Simple parameter passing might be enough


Gradual Adoption

You don't have to use DI everywhere. Start simple and add DI where it provides value:

# Level 1: Simple parameter passing (Pure DI)
def process_order(order, payment_gateway):
    payment_gateway.charge(order.total)

# Level 2: Constructor injection
class OrderProcessor:
    def __init__(self, payment_gateway):
        self.payment_gateway = payment_gateway

# Level 3: Interfaces/Abstractions
class OrderProcessor:
    def __init__(self, payment_gateway: PaymentGateway):  # Abstract type
        self.payment_gateway = payment_gateway

# Level 4: DI Container (for complex applications)
container = Container()
container.register(PaymentGateway, StripeGateway)
processor = container.resolve(OrderProcessor)


Sources:
- DataCamp - When to Use Dependency Injection (https://www.datacamp.com/tutorial/dependency-injection-python)
- Reddit - Do you do dependency injection without a framework? (https://www.reddit.com/r/Python/comments/179sbcv/do_you_do_dependency_injection_without_a_framework/)
- Hyperskill - Dependency Injection in Python (https://hyperskill.org/learn/step/32286)
-------------------------

Additional Resources

Official Documentation

- Python ABC Module (https://docs.python.org/3/library/abc.html)
- Python typing Module (https://docs.python.org/3/library/typing.html)
- Python Protocols (PEP 544) (https://peps.python.org/pep-0544/)
- dependency-injector Documentation (https://python-dependency-injector.ets-labs.org/)
- injector Documentation (https://injector.readthedocs.io/en/latest/)
- FastAPI Dependency Injection (https://fastapi.tiangolo.com/tutorial/dependencies/)
- pytest Fixtures (https://docs.pytest.org/en/stable/how-to/fixtures.html)
Tutorials and Guides

- Snyk - The Ultimate Guide to Dependency Injection in Python (https://snyk.io/blog/dependency-injection-python/)
- Toptal - Python Dependency Injection Tutorial (https://www.toptal.com/python/python-dependency-injection-design-pattern)
- DataCamp - Dependency Injection in Python (https://www.datacamp.com/tutorial/dependency-injection-python)
- YourDevOps - Python Dependency Injection: A Comprehensive Guide (https://www.yourdevops.org/python-dependency-injection)
- Towards Dev - Dependency Injection in Python (https://towardsdev.com/dependency-injection-in-python-2456dd37dfdb)
Framework-Specific Resources

- dependency-injector Tutorials (https://python-dependency-injector.ets-labs.org/tutorials/index.html)
- Snyk - Getting Started With DI Using the injector Library (https://snyk.io/blog/getting-started-with-di-in-python-using-the-injector-library/)
- Better Stack - Dependency injection in FastAPI (advanced) (https://betterstack.com/community/guides/scaling-python/fastapi-dependency-injection/)
Design Principles

- Dev.to - SOLID: Dependency Inversion Principle in Python (https://dev.to/alexmercedcoder/learning-solid-principles-through-python-part-5-dependency-inversion-principle-2n8i)
- Medium - SOLID Principles in Python (https://medium.com/@mudasirhaji/solid-principles-in-python-a-comprehensive-guide-e5a99b423df3)
- Dev.to - Why Use Abstractions in Dependency Injection (https://dev.to/dstarner/why-use-abstractions-in-dependency-injection-exploring-the-benefits-in-python-25mp)
Libraries and Frameworks

| Library | PyPI | GitHub |
|---------|------|--------|
| dependency-injector | [PyPI](https://pypi.org/project/dependency-injector/) | [GitHub](https://github.com/ets-labs/python-dependency-injector) |
| injector | [PyPI](https://pypi.org/project/injector/) | [GitHub](https://github.com/python-injector/injector) |
| punq | [PyPI](https://pypi.org/project/punq/) | [GitHub](https://github.com/bobthemighty/punq) |
| kink | [PyPI](https://pypi.org/project/kink/) | [GitHub](https://github.com/kodemore/kink) |
| lagom | [PyPI](https://pypi.org/project/lagom/) | [GitHub](https://github.com/meadsteve/lagom) |



-------------------------

Summary

Dependency Injection is a powerful design pattern that, when used appropriately, can significantly improve your Python code's:
- Testability: Easy to mock dependencies for unit testing
- Maintainability: Changes to dependencies don't ripple through your codebase  
- Flexibility: Swap implementations without modifying consuming code
- Readability: Dependencies are explicit and documented in constructors
Key Takeaways

- Start Simple: Use constructor injection before reaching for frameworks
- Depend on Abstractions: Use ABC or Protocols to define interfaces
- Use Constructor Injection for required dependencies
- Use Setter Injection for optional dependencies
- Create a Composition Root: Wire dependencies in one central place
- Choose the Right Tool: Use frameworks for complex applications, pure DI for simpler ones
- Test with DI: Leverage DI for easy mocking and fixture injection
Remember: Dependency Injection is a means to an end (better code), not an end in itself. Use it where it provides value, and keep things simple where complexity isn't needed.
-------------------------

Document last updated: 2025
Created for mid-level Python developers looking to understand and implement dependency injection patterns




File created successfully at /workspace/Python/DependencyInjection.md

I've created a comprehensive guide on Dependency Injection in Python. Let me verify the file was created correctly and show you a summary of what's included:
Table of Contents

- Introduction (#introduction)
- Core Concepts (#core-concepts)
- Why Use Dependency Injection? (#why-use-dependency-injection)
- Types of Dependency Injection (#types-of-dependency-injection)
- Implementing DI Without Frameworks (#implementing-di-without-frameworks)
- Python Abstractions for DI (#python-abstractions-for-di)
- DI Frameworks and Libraries (#di-frameworks-and-libraries)
- Framework-Specific DI (#framework-specific-di)
- Testing with Dependency Injection (#testing-with-dependency-injection)
- Best Practices and Anti-Patterns (#best-practices-and-anti-patterns)
- Real-World Examples (#real-world-examples)
- When to Use (and Not Use) DI (#when-to-use-and-not-use-di)
- Additional Resources (#additional-resources)
-------------------------

Introduction

Dependency Injection (DI) is a design pattern in software development that manages how components in your application get their dependencies (other objects they need to work). Instead of a class creating its own dependencies internally, dependencies are "injected" from outside, making the code more maintainable, flexible, and easier to test.
DI is part of a larger software design principle known as Inversion of Control (IoC), where the control of dependency management shifts from the components themselves to an external handler.
> **Key Insight**: While DI is less frequently used in Python than in languages like Java or C#, it offers powerful design benefits for creating maintainable and testable code.

Sources:
- Snyk - The Ultimate Guide to Dependency Injection in Python (https://snyk.io/blog/dependency-injection-python/)
- Toptal - Python Dependency Injection Tutorial (https://www.toptal.com/python/python-dependency-injection-design-pattern)
-------------------------

Core Concepts

Inversion of Control (IoC)

IoC is a principle that reshapes our approach to designing maintainable and scalable software. By shifting the responsibility for managing dependencies from components to an external container or framework, we achieve:
- Loose coupling between components
- Better separation of concerns
- Improved testability

# WITHOUT IoC - Class controls its own dependencies
class OrderService:
    def __init__(self):
        # The class creates its own dependency - tight coupling!
        self.payment_gateway = PayPalGateway()  # Hardcoded dependency
    
    def process_order(self, order):
        self.payment_gateway.charge(order.total)

# WITH IoC - Dependencies are provided from outside
class OrderService:
    def __init__(self, payment_gateway):
        # Dependency is injected - loose coupling!
        self.payment_gateway = payment_gateway  # Any payment gateway can be used
    
    def process_order(self, order):
        self.payment_gateway.charge(order.total)


Dependency Inversion Principle (DIP)

The Dependency Inversion Principle is the "D" in SOLID principles. It states:
- High-level modules should not depend on low-level modules. Both should depend on abstractions.
- Abstractions should not depend on details. Details should depend on abstractions.

from abc import ABC, abstractmethod

# ABSTRACTION - Both high-level and low-level modules depend on this
class NotificationService(ABC):
    @abstractmethod
    def send(self, message: str, recipient: str) -> None:
        pass

# LOW-LEVEL MODULE - Implements the abstraction
class EmailNotificationService(NotificationService):
    def send(self, message: str, recipient: str) -> None:
        # Email sending logic
        print(f"Sending email to {recipient}: {message}")

class SMSNotificationService(NotificationService):
    def send(self, message: str, recipient: str) -> None:
        # SMS sending logic
        print(f"Sending SMS to {recipient}: {message}")

# HIGH-LEVEL MODULE - Depends on abstraction, not implementation
class UserController:
    def __init__(self, notification_service: NotificationService):
</result>

The file has been created successfully. Let me show you the full structure by checking the file length:

<function_calls>
<invoke name="bash">
<parameter name="command">wc -l /workspace/Python/DependencyInjection.md && echo "---" && head -100 /workspace/Python/DependencyInjection.md | grep -E "^#{1,3} " | head -30</parameter>
</invoke>
</function_calls>
<result>
1572 /workspace/Python/DependencyInjection.md
---
# Comprehensive Guide to Dependency Injection in Python
## Table of Contents
## Introduction
## Core Concepts
### Inversion of Control (IoC)
### Dependency Inversion Principle (DIP)
## Why Use Dependency Injection?
### Benefits
### Without DI vs With DI
## Types of Dependency Injection
### 1. Constructor Injection (Most Common)
### 2. Setter Injection (Property Injection)
### 3. Method Injection (Interface Injection)
## Implementing DI Without Frameworks
### Simple Manual DI
### Using a Simple DI Container (DIY)
## Python Abstractions for DI
### 1. Abstract Base Classes (ABC)
### 2. Protocols (Structural Typing) - Python 3.8+
### ABC vs Protocol: When to Use Which?
## DI Frameworks and Libraries
### 1. dependency-injector (Most Popular)
### 2. injector Library
### 3. punq (Lightweight)
### 4. kink (Simple and Modern)
### 5. lagom
### Framework Comparison
## Framework-Specific DI
### FastAPI Dependency Injection
</parameter>
</invoke>
</function_calls>
<result>
1572 /workspace/Python/DependencyInjection.md
---
# Comprehensive Guide to Dependency Injection in Python
## Table of Contents
## Introduction
## Core Concepts
### Inversion of Control (IoC)
### Dependency Inversion Principle (DIP)
## Why Use Dependency Injection?
### Benefits
### Without DI vs With DI
## Types of Dependency Injection
### 1. Constructor Injection (Most Common)
### 2. Setter Injection (Property Injection)
### 3. Method Injection (Interface Injection)
## Implementing DI Without Frameworks
### Simple Manual DI
### Using a Simple DI Container (DIY)
## Python Abstractions for DI
### 1. Abstract Base Classes (ABC)
### 2. Protocols (Structural Typing) - Python 3.8+
### ABC vs Protocol: When to Use Which?
## DI Frameworks and Libraries
### 1. dependency-injector (Most Popular)
### 2. injector Library
### 3. punq (Lightweight)
### 4. kink (Simple and Modern)
### 5. lagom
### Framework Comparison
## Framework-Specific DI
### FastAPI Dependency Injection
</result>

I've created a comprehensive **1,572-line** guide on Dependency Injection in Python at `/workspace/Python/DependencyInjection.md`. Here's a summary of what's covered:

## Document Overview

### Sections Included:

1. **Introduction** - What DI is and why it matters in Python
2. **Core Concepts** - IoC and Dependency Inversion Principle (SOLID's "D")
3. **Why Use DI** - Benefits with comparison tables
4. **Types of DI** - Constructor, Setter, and Method injection with full code examples
5. **Manual DI Implementation** - How to do DI without any framework, including a DIY container
6. **Python Abstractions** - ABC vs Protocols comparison with examples
7. **DI Frameworks** - Coverage of 5 major libraries:
   - `dependency-injector` (most popular)
   - `injector` (Guice-inspired)
   - `punq` (lightweight)
   - `kink` (simple)
   - `lagom` (minimal config)
8. **Framework-Specific DI** - FastAPI, Flask, and Django examples
9. **Testing with DI** - Mock objects, pytest fixtures, container overrides
10. **Best Practices & Anti-Patterns** - What to do and what to avoid
11. **Real-World Examples** - E-commerce order processing & async application
12. **When to Use DI** - Decision framework
13. **Additional Resources** - Curated links to documentation and tutorials

### Key Features:
- ✅ All code examples have detailed comments explaining each step
- ✅ Sources are linked throughout the document
- ✅ Comparison tables for quick reference
- ✅ Practical real-world examples
- ✅ Both beginner-friendly explanations and advanced patterns
- ✅ Framework comparison chart to help you choose