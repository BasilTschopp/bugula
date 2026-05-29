## Python Code Conventions

### Project Structure

```
src/
├── adapters/      # External systems: database, browser, email, crypto
├── interfaces/    # GUI: windows, views, styles
├── models/        # Data classes and shared helpers
└── usecases/      # Business logic: test execution, recording, reading/writing
data/              # Database and encryption key (not committed to git)
docs/              # Documentation and conventions
```

### Code Style

Follows the [PEP 8 Style Guide](https://peps.python.org/pep-0008/).
