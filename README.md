# Encapsulate domain logic
- **Order class:**
  - In inside Order class we create order
```php
final class Order{

    public function __construct(
        private OrderId $orderId,
        private Client $client,
        private Status $status,
        private string $invoice
    ){}

    public function createOrder(CreateOrderCommand $command): self
    {
        return new self(
            OrderId::generateId(),
            new Client(
                new ClientId($command->clientId),
                new FullName($command->fistName, $command->lastName),
                new Address(
                    $command->state,
                    $command->city,
                    $command->street,
                    $command->zipCode
                ),
            ),
            $this->status = Status::ORDER_IS_ACCEPTED->value,
            $command->invoice
        );
    }
}
```
- **Class Client:**
  - This is class contains V.O ClientId, FullName, Address
```php
final class Client{
    public function __construct(
        private ClientId $clientId,
        private FullName $name,
        private Address $address,
    ){}
}
```
- **V.O ClientId:**
    - In this is V.O validate Ramsey/Uuid to must only Uuid::uuid7()
```php
final class ClientId{

    private UuidInterface $uuid;

    public function __construct(string $uuid)
    {
        $this->validateUuid($uuid);
        $this->uuid = Uuid::fromString($uuid);
    }

    public static function generateId(): string
    {
        return Uuid::uuid7()->toString();
    }

    public function __toString(): string
    {
        return $this->uuid->toString();
    }

    private function validateUuid(string $uuid): void
    {
        if (!Uuid::isValid($uuid)) {
            throw new InvalidArgumentException("Invalid UUID format from ClientId.");
        }

        if (substr($uuid, 14, 1) !== '7') {
            throw new InvalidArgumentException("Invalid UUID version from ClientId. Must be version 7.");
        }
    }
}
```
- **V.O OrderId:**
  - In this is V.O validate Ramsey/Uuid to must only Uuid::uuid7()
```php

final class OrderId{

    private UuidInterface $uuid;

    public function __construct(string $uuid)
    {
        $this->validateUuid($uuid);
        $this->uuid = Uuid::fromString($uuid);
    }

    public static function generateId(): string
    {
        return Uuid::uuid7()->toString();
    }

    public function __toString(): string
    {
        return $this->uuid->toString();
    }

    private function validateUuid(string $uuid): void
    {
        if (!Uuid::isValid($uuid)) {
            throw new InvalidArgumentException("Invalid UUID format from OrderId.");
        }

        if (substr($uuid, 14, 1) !== '7') {
            throw new InvalidArgumentException("Invalid UUID version from OrderId. Must be version 7.");
        }
    }
}
```
- **FullName:**
  - V.O
```php
final class FullName{

    public function __construct(
        private string $firstName,
        private string $lastName
    ){
        $this->validateFirstName($this->firstName);
        $this->validateLastName($this->lastName);
    }

    private function validateFirstName(string $firstName)
    {
        if (empty($firstName)) {
            throw new InvalidArgumentException("First name cannot be empty.");
        }

        if (!ctype_alpha($firstName)) {
            throw new InvalidArgumentException("First name must contain only alphabetic characters.");
        }

        $minLength = 2;
        $maxLength = 50;
        $nameLength = strlen($firstName);
        if ($nameLength < $minLength || $nameLength > $maxLength) {
            throw new InvalidArgumentException("First name must be between $minLength and $maxLength characters.");
        }
    }

    private function validateLastName(string $lastName)
    {
        if (empty($lastName)) {
            throw new InvalidArgumentException("Last name cannot be empty.");
        }

        if (!ctype_alpha($lastName)) {
            throw new InvalidArgumentException("Last name must contain only alphabetic characters.");
        }

        $minLength = 2;
        $maxLength = 50;
        $lastNameLength = strlen($lastName);
        if ($lastNameLength < $minLength || $lastNameLength > $maxLength) {
            throw new InvalidArgumentException("Last name must be between $minLength and $maxLength characters.");
        }
    }

}
```
- **Enum status**
```php
enum Status: string{
    case ORDER_IS_ACCEPTED = 'accepted';
    case ORDER_IS_REJECTED = 'rejected';
}
```

- **Command DTO**
  - In this case use CQRS or simple Command pattern
```php
final readonly class CreateOrderCommand{

    public function __construct(
        public string $clientId,
        public string $firstName,
        public string $lastName,
        public string $state,
        public string $city,
        public string $street,
        public string $zipCode,
        public string $invoice,
    ){}
}
```

```php
$command = new CreateOrderCommand(
    "018ccef6-aace-735e-bdd9-9884aa25860e",
    "John",
    "Doe",
    "California",
    "Los Angeles",
    "123 Main Street",
    "018ccef6-e367-70ae-ad47-cabf1728971e",
    "INV-2024001"
);

$order = Order::createOrder($command);
var_dump($order);
```