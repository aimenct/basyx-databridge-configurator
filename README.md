# BaSyx DataBridge Files Generator (Python Version)

🚀 **Automatic configuration files generator for BaSyx DataBridge synchronization**

This Python application automatically generates JSON configuration files for BaSyx DataBridge by reading Asset Administration Shell (AAS) submodels and creating the necessary protocol consumers, transformers, routes, and AAS server configurations.

## 🔄 How It Works

1. **Submodel Configuration**: Submodel IDs are defined in the `.env` file (DATABRIDGE_ID, DATABRIDGE_ID_AID, DATABRIDGE_ID_AIMC)
2. **AAS Information Retrieval**: The application connects to the BaSyx AAS server and downloads detailed information about the configured submodels
3. **Configuration Generation**: Based on the submodel properties and data types, the application automatically generates:
   - Kafka/OPC UA/HTTP consumer configurations with UUID-based unique identifiers
   - JSONata transformers for data mapping
   - Route definitions for data flow
   - AAS server connection settings
4. **File Output**: All configuration files are saved in the `databridge_config_files/` directory, ready to be used by BaSyx DataBridge

## 📋 AAS Examples

This project includes three comprehensive Asset Administration Shell (AAS) examples located in the `./aas/` directory, demonstrating different aspects of Industry 4.0 data integration:

### 🤖 Robot AAS (`./aas/Robot.aasx`)
- **Purpose**: Defines the digital twin of an industrial robot
- **Data Sources**: Robot joint states, positions, velocities, and operational status
- **Integration**: Maps robot sensor data from Kafka topics to standardized AAS properties

### 🔄 DataBridge AAS (`./aas/Databridge.aasx`)
- **Purpose**: Contains communication protocols, data mapping schemas, and transformation rules for data synchronization
- **Standard Submodels**: Includes AIMC (Asset Interface Mapping Configuration), AID (Asset Interface Description), and other submodels for interoperability
- **Protocol Support**: Kafka, OPC UA, HTTP protocol definitions
- **Data Flow**: Defines how data flows from Kafka topics to HTTP AAS server endpoints and integrates with standard submodels for seamless data exchange

### 🌐 Protocol Integration Architecture
The DataBridge AAS serves as the central coordination point:

```
Kafka Topics          DataBridge Python       BaSyx HTTP AAS Endpoints
┌─────────────────┐   ┌─────────────────┐     ┌────────────────────────┐
│ MPCM.joint_tool │──►│ Protocol Mapper │────►│ Robot AAS Properties   │
│ _states         │   │                 │     │                        │
└─────────────────┘   │                 │     └────────────────────────┘
┌─────────────────┐   │                 │     ┌────────────────────────┐
│ PAM.body_track  │──►│ Data Transform  │────►│ Operator AAS Properties│
└─────────────────┘   └─────────────────┘     └────────────────────────┘
```
## 🔧 System Requirements

### Docker Execution (Recommended)
- **Docker** 20.10+  
- **Docker Compose** 1.29+  
- **Minimum 2GB RAM** for containers

### Local Python Execution
- **Python 3.11+**
- **pip** or **conda** for package management
- **Network access** to the BaSyx server specified in `.env`

## 🚀 Execution

### 🐳 Docker Execution (Recommended)

#### Docker services

This Docker Compose setup includes a complete Industry 4.0 data integration stack:

| Service | Purpose | Port | Description |
|---------|---------|------|-------------|
| **kafka-broker** | 📡 Message Broker | 9092, 29092 | Apache Kafka message streaming platform |
| **aas-env** | 🏭 AAS Server | 8081 | BaSyx AAS Environment server for Asset Administration Shells |
| **databridge-configurator** | 📋 Config Generator | - | Python-based generator for BaSyx DataBridge configuration files from AAS submodels |
| **databridge** | 🔄 Data Bridge | - | BaSyx DataBridge service that consumes Kafka data and updates AAS properties |
| **aas-web-ui** | 🖥️ Web Interface | 3000 | BaSyx AAS Web UI for monitoring and managing Asset Administration Shells |
| **robot-data-generator** | 🤖 Robot Simulator | - | Publishes realistic robot joint state data to Kafka topic `MPCM.joint_tool_states` |

### 🔄 Service Interaction Flow
```
┌─────────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│ Data Generators     │───►│ Kafka Broker    │───►│ BaSyx DataBridge│───►│ AAS Environment │───►│ AAS Web UI          │
│ (Robot Simulator)   │    │ (Topics)        │    │ (Config Files)  │    │ (HTTP API)      │    │ (Monitor/Control)   │
└─────────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────────┘

```

### 🚀 Complete System Startup
```bash
# Start all services in the correct order
docker-compose up -d

# Start databridge services
docker-compose --profile databridge up -d

# Check service health
docker-compose ps

# View logs from specific services
docker-compose logs -f databridge-configurator
docker-compose logs -f robot-data-generator
```

> **⚠️ IMPORTANT NOTE - Configuration Demonstration:**
> 
> The `Databridge.aasx` file contains **intentionally incorrect endpoints** to demonstrate the software's configuration generation functionality. To run the system correctly, you must:
> 
> 1. **Access the AAS Web UI** at http://localhost:3000
> 2. **Navigate to the DataBridge AAS** and locate the submodels
> 3. **Fix the Kafka Submodel**:
>    - Go to: **DataBridge** → **AssetInterfacesDescription** → **Kafka** → **EndpointMetadata** → **base**
>    - Change from `"kafka-broke"` to `"kafka-broker:29092"`
> 4. **Fix the HTTP Submodel**:
>    - Go to: **DataBridge** → **AssetInterfacesDescription** → **HTTP** → **EndpointMetadata** → **base**
>    - Change from `"aas-env"` to `"http://aas-env:8081"`
> 
> These corrections will trigger the configuration regeneration process and demonstrate how the system automatically updates when AAS submodel properties change.

### 🐍 Python Execution (Local Development)
```bash
# Clone the repository
git clone http://gitlab.aimen.local/iiot/converging/basyx-databridge-files-generator_python.git
cd basyx-databridge-files-generator_python

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.template .env
# Edit .env with your configuration

# Run the generator
python main.py
```

## ⚙️ Configuration

### Environment Variables

Copy the template and configure your environment:

```bash
# Copy template
cp .env.template .env

# Edit configuration
nano .env
```

The `.env` file contains the BaSyx server configurations and submodel identifiers:

```properties
BASE_URL=http://aas-env:8081
DATABRIDGE_ID=aHR0cHM6Ly93d3cuYWltZW4uZXMvYWFzL3h4bHBpbG90ZmFjdG9yeS9kYXRhYnJpZGdl
DATABRIDGE_ID_AID=aHR0cHM6Ly93d3cuYWltZW4uZXMvc20vYWlkL3h4bGlwaWxvdGZhY3RvcnkvZGF0YWJyaWRnZQ==
DATABRIDGE_ID_AIMC=aHR0cHM6Ly93d3cuYWltZW4uZXMvc20vYWltYy94eGxpcGlsb3RmYWN0b3J5L2RhdGFicmlkZ2U=
OUTPUT_DIR=databridge_config_files
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BASE_URL` | BaSyx AAS server base URL | `"http://aas-env:8081"` |
| `DATABRIDGE_ID` | Main databridge submodel ID (base64) | `"aHR0cHM6Ly9...="` |
| `DATABRIDGE_ID_AID` | Asset Interfaces Description ID | `"eHhscGlsb3Q...="` |
| `DATABRIDGE_ID_AIMC` | Asset Interface Mapping Config ID | `"eHhscGlsb3Q...="` |
| `OUTPUT_DIR` | Output directory for generated files | `"databridge_config_files"` |

## 📁 Generated Files for DataBridge service

Files are created in `databridge_config_files/`:

```
databridge_config_files/
├── aasserver.json           # AAS Server configurations with UUID identifiers
├── routes.json              # Routes configurations with UUID-based data sources
├── jsonatatransformer.json  # JSONata transformers with UUID identifiers
├── kafkaconsumer.json       # Kafka Consumer configurations (if applicable)
├── opcuaconsumer.json       # OPC UA Consumer configurations (if applicable)
├── mqttconsumer.json        # MQTT Consumer configurations (if applicable)
└── *.json                   # Individual variable transformation files
```

## � Project Structure

```
basyx-databridge-files-generator-python/
├── main.py                          # 🚀 Main entry point
├── service/
│   ├── submodels.py                 # 🔍 AAS submodel information handler
│   └── registryConnect.py           # 🌐 AAS registry connection utilities
├── databridge_configurator/        # ⚙️ Configuration generation logic
│   ├── configurationFiles.py       # 📝 Main configuration file generator
│   ├── topics.py                    # 🎯 Topic data structures
│   └── models/                      # 📦 Data model classes
│       ├── __init__.py
│       ├── routes_schema.py         # 🛤️ Route configuration schema
│       ├── protocol_schema.py       # 📡 Protocol consumer schemas (Kafka, OPC UA, MQTT)
│       ├── jsonatatransformer_schema.py # 🔄 JSONata transformer schema
│       └── aasServerJson_schema.py  # 🏭 AAS server configuration schema
├── aas/                             # 📁 Example AAS files
│   ├── Databridge.aasx             # 🔄 DataBridge AAS with protocol configurations
│   └── Robot.aasx                  # 🤖 Robot AAS with joint state definitions
├── databridge_config_files/        # 📁 Output directory for generated files (git ignored)
├── logs/                            # 📝 Application logs (git ignored)
├── .env                             # 🔑 Environment variables (git ignored)
├── .env.template                    # 🔑 Environment variable template
├── .gitignore                       # 🚫 Git ignore rules
├── .dockerignore                    # 🐳 Docker ignore rules
├── Dockerfile                       # 🐳 Docker build instructions
├── docker-compose.yml               # 🐳 Docker Compose configuration
├── requirements.txt                 # 📋 Python dependencies
└── README.md                        # 📖 Project documentation
```

## 🎲 Data Generators

This project integrates with complementary data generators that simulate realistic data sources for testing and development purposes:

### 🤖 Robot Data Generator
- **Purpose**: Simulates industrial robot joint states and operational data
- **Data Types**: Joint positions, velocities, efforts, and operational states
- **Protocol**: Kafka messaging to topic `MPCM.joint_tool_states`
- **Format**: ROS-compatible sensor_msgs/JointState format
- **Features**: Realistic movement patterns, configurable publish rates

### 📡 Integration with BaSyx DataBridge
The generated data flows through the following pipeline:
1. **Data Generators** → Kafka Topics (`MPCM.joint_tool_states`)
2. **BaSyx DataBridge** → Consumes Kafka data using Python-generated configuration files
3. **AAS Server** → Exposes data through standardized Asset Administration Shell interface

### ⚙️ Generator Configuration
Each generator can be configured through environment variables:
- **KAFKA_BROKER**: Kafka broker connection string
- **PUBLISH_INTERVAL**: Data publishing frequency (seconds)
- **TOPIC_JOINT_STATES**: Robot joint states topic name

## 🔧 Development

### Setting up development environment

```bash
# Clone repository
git clone http://gitlab.aimen.local/iiot/converging/basyx-databridge-files-generator_python.git

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# or
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.template .env
# Edit .env with your configuration

# Run the generator
python main.py
```

### Code Architecture

- **Models**: Pydantic-based data classes representing configuration schemas
- **Services**: Business logic for submodel processing and AAS communication
- **Configurator**: File generation logic with UUID-based unique identifiers
- **Utils**: Helper functions for HTTP requests and data transformation

## 📚 Dependencies

- `basyx-python-sdk`: Official BaSyx Python SDK for AAS operations
- `loguru`: Advanced logging with structured output
- `lxml`: XML processing for AAS file handling
- `requests`: HTTP client for BaSyx API communication
- `python-dotenv`: Environment variable management
- `pydantic`: Data validation and settings management
- `jsonpath_ng`: JSONPath expressions for data querying

## 🙏 Acknowledgments

This project is developed as part of the **CONVERGING** project, which aims to develop, deploy, validate, and promote smart and reconfigurable production systems including multiple autonomous agents (collaborative robots, AGVs, humans) that are able to act in diverse production environments.

🌐 **Project Website**: [https://www.converging-project.eu/](https://www.converging-project.eu/)

🇪🇺 **Funding**: The CONVERGING project is co-funded by the European Union's Horizon Europe Research & Innovation Programme under Grant N° 101058521.

We acknowledge the support and contributions of all CONVERGING project partners and collaborators who have made this work possible.

## 📄 License

This project is licensed under review.

---

**🚀 Ready to streamline your BaSyx DataBridge configuration? Get started now!**

```bash
# Docker execution (recommended)
docker-compose up -d
docker-compose --profile databridge up -d

# Or local Python execution
python main.py
```
