# AI Workshop - OpenEdge ABL Project

This project demonstrates the ABL Business Entity Architecture pattern for OpenEdge applications.

## Architecture

The project follows a layered architecture pattern:

- **UI Layer**: Windows and forms for user interaction
- **Business Entity Layer**: Data access, business rules, and validation
- **Database Layer**: Persistent storage (sports2000 database)

## Key Components

- **EntityFactory**: Singleton factory for managing business entity instances
- **Business Entities**: CustomerEntity, ItemEntity - encapsulate data operations
- **Datasets**: Temp-table definitions for data transfer
- **UI Windows**: CustomerWin.w, ItemWin.w - refactored to use business entities

## Documentation

See [doc/business-entity-pattern.md](doc/business-entity-pattern.md) for detailed documentation on the Business Entity Architecture pattern.

## Database Schema

The project uses the sports2000 database. See [dump/sports2000.df](dump/sports2000.df) for the database schema.

## Development

This project uses OpenEdge ABL 12.8.