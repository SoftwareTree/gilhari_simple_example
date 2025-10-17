> **Note:** This file is written in Markdown and is best viewed with a Markdown viewer (e.g., GitHub, GitLab, VS Code, or a dedicated Markdown reader). Viewing it in a plain text editor may not render the formatting as intended.

Copyright (c) 2025 Software Tree

# Gilhari Simple Example

> **Basic example demonstrating RESTful CRUD operations for JSON objects with user-specified IDs**

Gilhari is a Docker-compatible microservice framework that provides RESTful Object-Relational Mapping (ORM) functionality for JSON objects with any relational database.

Remarkably, Gilhari automates REST APIs (POST, GET, PUT, DELETE, etc.) handling, JSON CRUD operations, and database schema setup — **no manual coding required**.

## About This Example

This repository contains a simple, straightforward example showing how to use Gilhari to create a RESTful microservice for persisting JSON objects with basic CRUD operations, filtering, and aggregate queries using user-specified primary keys.

The example uses the base Gilhari docker image (softwaretree/gilhari) to easily create a new docker image (gilhari_simple_example) that can run as a RESTful microservice (server) to persist app specific JSON objects.

This example can be used **standalone as a RESTful microservice** or optionally with the ORMCP Server.

**Related:**
- Main ORMCP Server: [https://github.com/SoftwareTree/ormcp-server](https://github.com/SoftwareTree/ormcp-server)
- Autoincrement variant: [gilhari_autoincrement_example](https://github.com/SoftwareTree/gilhari_autoincrement_example) - Similar but with database-generated IDs

**Note:** This example is also included in the Gilhari SDK distribution. If you have the SDK installed, you can use it directly from the `examples/gilhari_simple_example` directory without cloning.

## Example Overview

The example showcases a JSON object model with one type of object: **Employee** (or **JSON_Employee**)

**Object Model Overview:**
- **JSON_Employee**: Simple employee object with user-specified ID
- **Attributes**: id (int - user-specified), name (string), exempt (boolean), compensation (double), DOB (long/milliseconds)
- **Database Table**: Employee with column `salary` for the compensation attribute
- **Database Column**: `salary` (mapped from `compensation` attribute)

### Employee Object Structure
```json
{
  "id": 39,
  "name": "John39",
  "compensation": 54039.0,
  "exempt": true,
  "DOB": 381484800390
}
```

**Features Demonstrated:**
- Basic CRUD operations (Create, Read, Update, Delete)
- User-specified primary keys (you provide the ID value)
- Querying with filters (e.g., `exempt=0`, `exempt=1`)
- Aggregate queries (COUNT)
- Batch operations (inserting multiple objects at once)
- Advanced projections using `operationDetails` parameter
- Object model introspection via `getObjectModelSummary` endpoint
- Health check endpoint
- Column mapping (compensation → salary in database)

### Comparison with Other Examples

**gilhari_simple_example vs gilhari_autoincrement_example:**
- **Simple example** (this): You specify the `id` value when creating employees
- **Autoincrement example**: Database automatically generates the `id` value

Both examples use similar Employee objects but differ in primary key management strategy.

## Project Structure

```
gilhari_simple_example/
├── src/                           # Container domain model classes
│   └── com/softwaretree/...      # JSON_Employee.java and base classes
├── config/                        # Configuration files
│   ├── gilhari_simple_example.jdx # ORM specification
│   └── classnames_map_example.js
├── bin/                           # Compiled .class files
├── Dockerfile                     # Docker image definition
├── gilhari_service.config         # Service configuration
├── compile.cmd / .sh              # Compilation scripts
├── build.cmd / .sh                # Docker build scripts
├── run_docker_app.cmd / .sh       # Docker run scripts
├── curlCommands.cmd / .sh         # API testing scripts
└── curlCommandsPopulate.cmd / .sh # Sample data population scripts
```

## Source Code
The `src` directory contains the declarations of the underlying shell (container) classes (e.g., JSON_Employee) that are used to define the object-relational mapping (ORM) specification for the corresponding conceptual domain-specific JSON object model classes:

- **JSON_Employee class**: Simple shell (container) class (.java file) corresponding to the domain-specific JSON object model class (Container domain model class)
- **JDX_JSONObject**: Base class of the container domain model classes for handling persistence of domain-specific JSON objects
- **Container domain model classes**: Only need to define two constructors, with most processing handled by the JDX_JSONObject superclass

**Note:** Gilhari does not require any explicit programmatic definitions (e.g., ES6 style JavaScript classes) for domain-specific JSON object model classes. It handles the data of domain-specific JSON objects using instances of the container domain model classes and the ORM specification.

## Configurations

A declarative ORM specification for the domain-specific JSON object model classes and their attributes is defined in `config/gilhari_simple_example.jdx` using the container domain model classes. This file defines the mappings between JSON objects and database tables.

**Key points:**
- Update the database URL and JDBC driver in this file according to your setup
- See `JDX_DATABASE_JDBC_DRIVER_Specification_Guide` (.md or .html) for guides on configuring different databases
- The container domain model class (JSON_Employee) corresponding to the conceptual domain-specific JSON object model class is defined as a subclass of the JDX_JSONObject class
- Appropriate mappings for the domain-specific JSON object model class are defined in the ORM specification file using the corresponding container domain model class
- **Column mapping**: The `compensation` attribute is mapped to the `salary` column in the database
- **Date handling**: DOB is stored as a long (milliseconds) and mapped to DATE type in the database

For comprehensive details on defining and using container classes and the ORM specification for JSON object models, refer to the **"Persisting JSON Objects"** section in the JDX User Manual.

### Docker Configuration

The `Dockerfile` builds a RESTful Gilhari microservice using:
- Base Gilhari image (softwaretree/gilhari)
- Compiled domain model (.class) files
- Configuration files including the ORM specification and a JDBC driver

### Service Configuration

The `gilhari_service.config` file specifies runtime parameters for the RESTful Gilhari microservice:

```json
{
  "gilhari_microservice_name": "gilhari_simple_example",
  "jdx_orm_spec_file": "./config/gilhari_simple_example.jdx",
  "jdbc_driver_path": "/node/node_modules/jdxnode/external_libs/sqlite-jdbc-3.50.3.0.jar",
  "jdx_debug_level": 5,
  "jdx_force_create_schema": "true",
  "jdx_persistent_classes_location": "./bin",
  "classnames_map_file": "config/classnames_map_example.js",
  "gilhari_rest_server_port": 8081
}
```

#### Service Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `gilhari_microservice_name` | Optional name to identify this Gilhari microservice. The name is logged on console during start up | - |
| `jdx_orm_spec_file` | Location of the ORM specification file containing mapping for persistent classes | - |
| `jdbc_driver_path` | Path to the JDBC driver (.jar) file. SQLite driver included by default | - |
| `jdx_debug_level` | Debug output level (0-5). 0 = most verbose, 5 = minimal. Level 3 outputs all SQL statements | 5 |
| `jdx_force_create_schema` | Whether to recreate database schema on each run. `true` = useful for development, `false` = create only once | false |
| `jdx_persistent_classes_location` | Root location for compiled persistent (Container domain model) classes. Can be a directory (e.g., ./bin) or a JAR file path. Used as a Java CLASSPATH  | - |
| `classnames_map_file` | Optional JSON file that can map names of container domain model classes to (simpler) object class (type) names (e.g., by omitting a package name) to simplify REST URL| - |
| `gilhari_rest_server_port` | Port number for the RESTful service. This port number may be mapped to different port number (e.g., 80) by a docker run command. | 8081 |


## Build Files
- `compile.cmd` / `compile.sh`: Compiles the container domain model classes
- `sources.txt`: Lists the names of the container domain model class source (.java) files for compilation
- `build.cmd` / `build.sh`: Creates the Gilhari Docker image (gilhari_simple_example) using the local Dockerfile

**Note**: Compilation targets JDK version 1.8, which is compatible with the current Gilhari version.

## Quick Start

### For Quick Evaluation (No SDK Required)
If you just want to see this example in action without modifications:

1. **Clone this repository** (pre-compiled classes included)
2. **Install Docker**
3. **Build and run** (skip compilation step)

### For Development and Customization
If you want to modify the object model or create your own Gilhari microservices:

1. **Gilhari SDK**: Download and install from [https://softwaretree.com](https://softwaretree.com)
2. **JX_HOME environment variable**: Set to the root directory of your Gilhari SDK installation
3. **Java Development Kit (JDK 1.8+)** for compilation
4. **Docker** installed on your system

**Note:** The Gilhari SDK contains necessary libraries (JARs) and base classes required for compiling container domain model classes. While pre-compiled `.class` files are included in this repository for immediate use, you'll need the SDK to make any modifications to the object model or to create your own Gilhari microservices.

## Build and Run

### Option 1: Quick Run (Using Pre-compiled Classes)

**Skip compilation** and go straight to Docker:

```bash
# Windows
build.cmd
run_docker_app.cmd

# Linux/Mac
./build.sh
./run_docker_app.sh
```

### Option 2: Compile and Run (For Modifications)

**If you've made changes to the source code:**

1. **Ensure JX_HOME is set** to your Gilhari SDK installation directory

2. **Compile the classes**:
   ```bash
   # Windows
   compile.cmd
   
   # Linux/Mac
   ./compile.sh
   ```

3. **Build and run the Docker container**:
   ```bash
   # Windows
   build.cmd
   run_docker_app.cmd
   
   # Linux/Mac
   ./build.sh
   ./run_docker_app.sh
   ```

## REST API Usage

Once running, access the Gilhari microservice at:

```
http://localhost:<port>/gilhari/v1/:className
```

**Example endpoints:**
```
http://localhost:80/gilhari/v1/Employee
http://localhost:80/gilhari/v1/health/check
http://localhost:80/gilhari/v1/getObjectModelSummary/now
```

### Supported HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| GET | Retrieve objects | `GET /gilhari/v1/Employee` |
| POST | Create objects | `POST /gilhari/v1/Employee` |
| PUT | Update objects | `PUT /gilhari/v1/Employee` |
| PATCH | Partial update | `PATCH /gilhari/v1/Employee` |
| DELETE | Delete objects | `DELETE /gilhari/v1/Employee` |

### Example Operations

**Health Check:**
```bash
curl -X GET "http://localhost:80/gilhari/v1/health/check"
```

**Get Object Model Summary:**
```bash
curl -X GET "http://localhost:80/gilhari/v1/getObjectModelSummary/now"
```

**Create an Employee (with user-specified ID):**
```bash
curl -X POST http://localhost:80/gilhari/v1/Employee \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "id": 39,
      "name": "John39",
      "compensation": 54039.0,
      "exempt": true,
      "DOB": 381484800390
    }
  }'
```

**Create Multiple Employees:**
```bash
curl -X POST http://localhost:80/gilhari/v1/Employee \
  -H "Content-Type: application/json" \
  -d '{
    "entity": [
      {
        "id": 40,
        "name": "Mike40",
        "compensation": 54040.0,
        "exempt": false,
        "DOB": 381484800400
      },
      {
        "id": 41,
        "name": "Mary41",
        "compensation": 54041.0,
        "exempt": true,
        "DOB": 381484800410
      }
    ]
  }'
```

**Query Non-Exempt Employees:**
```bash
curl -X GET "http://localhost:80/gilhari/v1/Employee?filter=exempt=0" \
  -H "Content-Type: application/json"
```

**Count Exempt Employees:**
```bash
curl -X GET "http://localhost:80/gilhari/v1/Employee/getAggregate?attribute=id&aggregateType=COUNT&filter=exempt=1" \
  -H "Content-Type: application/json"
```

**Query with Projections (Selected Attributes Only):**
```bash
curl -G "http://localhost:80/gilhari/v1/Employee" \
  --data-urlencode "filter=exempt=0" \
  --data-urlencode 'operationDetails=[{"opType": "projections", "projectionsDetails": [{"type": "Employee", "attribs": ["name", "id", "exempt"]}]}]' \
  -H "Content-Type: application/json"
```

**Delete Exempt Employees:**
```bash
curl -X DELETE "http://localhost:80/gilhari/v1/Employee?filter=exempt=1"
```

### Testing the API

**Comprehensive test scripts:**

1. **curlCommands.cmd / .sh** - Complete demonstration of CRUD operations

   Demonstrates:
   - Health check endpoint
   - Getting object model summary
   - Creating single and multiple Employee objects with user-specified IDs
   - Querying with boolean filters (exempt=0, exempt=1)
   - Aggregate operations (COUNT)
   - Filtered deletion
   - Complete cleanup

2. **curlCommandsPopulate.cmd / .sh** - Data population with projections

   Demonstrates:
   - Populating sample Employee data
   - Boolean filtering for exempt/non-exempt employees
   - Aggregate queries for specific groups
   - **Projections** using `operationDetails` parameter for selecting specific attributes

Run the scripts to generate a `curl.log` file with all responses:
```bash
# Windows
curlCommands.cmd
curlCommandsPopulate.cmd

# Linux/Mac
chmod +x curlCommands.sh curlCommandsPopulate.sh
./curlCommands.sh
./curlCommandsPopulate.sh

# Custom port
curlCommands.cmd 8899
curlCommandsPopulate.sh 8899
```

The scripts will create a `curl.log` file with all the API responses, demonstrating Employee object management with user-specified IDs.

**Note:** The **`operationDetails`** parameter in a query allows you to fine-tune query operations with operational directives similar to **GraphQL** capabilities. It accepts a JSON array containing one or more operation directives that refine the shape and scope of returned objects. For more details, see `operationDetails_doc.md`.

**Other options:**
- **Postman**: Import the endpoints for interactive testing
- **Browser**: Access GET endpoints directly
- **Any REST Client**: Standard HTTP methods work with any REST client
- **ORMCP Server** (optional): Use ORMCP Server tools for AI-powered interactions

## Using with ORMCP Server (Optional)

This Gilhari microservice can be used with the ORMCP Server for AI-powered database interactions:

1. **Start this Gilhari microservice** (as shown in Quick Start)
2. **Configure ORMCP Server** to connect to this microservice endpoint
3. **Use natural language** to query and manipulate Employee objects through the ORMCP Server

The ORMCP Server can automatically discover the object model via the `getObjectModelSummary` endpoint.

For more information on ORMCP Server, visit: [https://github.com/SoftwareTree/ormcp-server](https://github.com/SoftwareTree/ormcp-server)

## Development Tools

### Docker Container Access
Shell into a running container:
```bash
# Find container ID
docker ps

# Access container
docker exec -it <container-id> bash
```

### View Logs
```bash
docker logs <container-id>
```

### Stop Container
```bash
docker stop <container-id>
```

## Additional Resources

- **JDX User Manual**: "Persisting JSON Objects" section for detailed ORM specification documentation
- **Gilhari SDK Documentation**: The SDK available for download at [https://softwaretree.com](https://softwaretree.com)
- **ORMCP Server**: Main repository at [https://github.com/SoftwareTree/ormcp-server](https://github.com/SoftwareTree/ormcp-server)
- **Database Configuration Guide**: See `JDX_DATABASE_JDBC_DRIVER_Specification_Guide.md`
- **operationDetails Documentation**: See `operationDetails_doc.md` for GraphQL-like query capabilities
- **Autoincrement variant**: [gilhari_autoincrement_example](https://github.com/SoftwareTree/gilhari_autoincrement_example)

## Platform Notes

Script files are provided for both Windows (`.cmd`) and Linux/Mac (`.sh`). 

**Linux/Mac users**: Make scripts executable before running:
```bash
chmod +x *.sh
```

## Troubleshooting

### Common Issues

**Problem**: Docker image build fails
- **Solution**: Ensure the base Gilhari image is pulled: `docker pull softwaretree/gilhari`

**Problem**: Compilation errors
- **Solution**: Verify JDK 1.8+ is installed and JX_HOME environment variable is set correctly

**Problem**: Port 80 already in use
- **Solution**: Modify `run_docker_app` script to use a different port (e.g., `-p 8080:8081`)

**Problem**: Database connection errors
- **Solution**: Check `config/gilhari_simple_example.jdx` for correct database URL and JDBC driver path

**Problem**: Duplicate ID errors when creating employees
- **Solution**: In this example, you must provide unique ID values. Each employee must have a different `id`. If you want automatic ID generation, use the gilhari_autoincrement_example instead

**Problem**: Boolean filter not working (exempt=0 or exempt=1)
- **Solution**: Boolean values in filters use 0 for false and 1 for true in SQL-style syntax

## Support

For issues or questions:
- **ORMCP Server issues**: [https://github.com/SoftwareTree/ormcp-server/issues](https://github.com/SoftwareTree/ormcp-server/issues)
- **This example**: [https://github.com/SoftwareTree/gilhari_simple_example/issues](https://github.com/SoftwareTree/gilhari_simple_example/issues)
- **Gilhari SDK**: Contact support at [gilhari_support@softwaretree.com](mailto:gilhari_support@softwaretree.com)

## License
This example code is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

**Important:** This license applies ONLY to the example code in this repository. The Gilhari software (including the softwaretree/gilhari Docker image and Gilhari SDK) and the embedded JDX ORM software are proprietary products owned by Software Tree.

The Gilhari Docker image includes an evaluation license for testing purposes. For production use or licensing beyond the evaluation period, please visit [https://www.softwaretree.com](https://www.softwaretree.com) or contact [gilhari_support@softwaretree.com](mailto:gilhari_support@softwaretree.com).

---

**Ready to try it?** Start with the [Quick Start](#quick-start) section above!