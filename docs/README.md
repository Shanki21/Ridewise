# RideWise

A console-based ride-sharing system (Uber/Ola style) written in **core Java**. It is built to demonstrate clean low-level design: the Strategy pattern, composition over inheritance, SOLID principles, and a clear separation of layers.

## Features

- Register riders and drivers
- View available drivers
- Request a ride, matched to a driver by a pluggable **matching strategy**
  - Nearest driver
  - Least active driver (fewest completed rides, closer driver breaks ties)
- Complete a ride, priced by a pluggable **fare strategy**
  - Default fare: 50 base + 12 per km
  - Peak hour fare: wraps another fare strategy and applies a multiplier
- Track the ride lifecycle: `REQUESTED`, `ASSIGNED`, `COMPLETED`, `CANCELLED`
- Invalid input never crashes the program

## Requirements

- JDK 17 or newer
- No external libraries or frameworks

## How to Run

**From the command line** (Windows cmd, from the project root):

```bat
dir /s /b src\*.java > sources.txt
javac -d out @sources.txt
java -cp out com.airtribe.ridewise.Main
```

**From IntelliJ IDEA:** open the project folder, mark `src` as the Sources Root, open `Main.java` and click the green run arrow next to `main`.

## Example Session

```
===== RideWise =====
1. Add Rider
2. Add Driver
3. View Available Drivers
4. Request Ride
5. Complete Ride
6. View Rides
7. Exit
Choose an option: 4
Rider id: 1
Trip distance in km: 10
Ride booked: Ride{id=1, rider=Asha, driver=Ravi, distance=10.0, status=ASSIGNED}

Choose an option: 5
Ride id: 1
Ride completed. Receipt{rideId=1, amount=170.00, at=2026-10-07T19:39:37.814636600}
```

Timestamps vary from run to run.

## Project Structure

```
src/com/airtribe/ridewise/
├── Main.java              console menu and wiring (composition root)
├── model/                 Rider, Driver, Ride, FareReceipt, Location, RideStatus, VehicleType
├── strategy/              RideMatchingStrategy, FareStrategy and their implementations
├── service/               RiderService, DriverService, RideService
├── exception/             NoDriverAvailableException
└── util/                  IdGenerator
docs/                      design documents
```

## Design Highlights

- **Strategy pattern:** `RideService` depends on the `RideMatchingStrategy` and `FareStrategy` interfaces. The concrete rules are chosen in `Main` and injected through the constructor.
- **Open for extension:** a new matching or pricing rule is a new class. No existing class changes.
- **Composition over inheritance:** `PeakHourFareStrategy` wraps any `FareStrategy` instead of extending `DefaultFareStrategy`.
- **Entities protect their own rules:** `Ride` rejects invalid status changes, so no caller can corrupt a ride.
- **Layering:** `Main` calls services, services call strategies and models, and models know nothing about the layers above.

## Changing the Strategies

The strategies are chosen once at startup in `Main.java`. For example, to use least-active matching with peak-hour pricing, change the two lines that build `RideService`:

```java
new LeastActiveDriverStrategy(),
new PeakHourFareStrategy(new DefaultFareStrategy(), 1.5)
```

## Adding a New Strategy

1. Create a class in `strategy/` that implements `RideMatchingStrategy` or `FareStrategy`.
2. Pass it to the `RideService` constructor in `Main`.

No other file needs to change.

## Documentation

| Document | Contents |
|----------|----------|
| [Requirements](docs/Requirements.md) | Functional and non-functional requirements, rules, assumptions |
| [Class Model](docs/Class_Model.md) | Packages, classes and the class diagram |
| [SOLID Reflection](docs/SOLID_Reflection.md) | How each principle appears in the code, and the trade-offs |
| [Object Relationships](docs/Object_Relationships.md) | Association, composition, lifecycles and sequence diagrams |

## Assumptions and Limitations

- Data is stored in memory only and is lost when the program exits.
- Locations are 2D points, and the trip distance is entered by the user.
- A ride is cancelled only when no driver is available.
- Out of scope: payments, persistence, real maps, ratings, multithreading.

See [Requirements](docs/Requirements.md) for the full list.
