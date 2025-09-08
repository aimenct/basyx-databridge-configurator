# BaSyx DataBridge Files Generator

🚀 **Automatic configuration files generator for BaSyx DataBridge synchronization**

This application automatically generates JSON configuration files for BaSyx DataBridge by reading Asset Administration Shell (AAS) submodels and creating the necessary protocol consumers, transformers, routes, and AAS server configurations.

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

This project includes 2 Asset Administration Shell (AAS) examples located in the `./aas/` directory, demonstrating different aspects of Industry 4.0 data integration:

### 🤖 Robot AAS (`./aas/Robot.aasx`)
- **Purpose**: Defines the digital twin of an industrial robot
- **Data Sources**: Robot joint states, positions, velocities, and operational data
- **Integration**: Maps robot data from Kafka topics to standardized AAS properties

### 🔄 DataBridge AAS (`./aas/Databridge.aasx`)
- **Purpose**: Contains communication protocols, data mapping schemas, and transformation rules for data synchronization
- **Standard Submodels**: Includes AIMC (Asset Interface Mapping Configuration), AID (Asset Interface Description), and other submodels for interoperability
- **Protocol Support**: Kafka, OPC UA, HTTP protocol definitions
- **Data Flow**: Defines how data flows from Kafka topics to HTTP AAS server endpoints and integrates with standard submodels for data exchange

### 🌐 Protocol Integration Architecture
The DataBridge AAS serves as the central coordination point:

```
Kafka Topics          DataBridge Configurator       BaSyx HTTP AAS Endpoints
┌─────────────────┐   ┌─────────────────┐     ┌────────────────────────┐
│ MPCM.joint_tool │──►│ Protocol Mapper │────►│ Robot AAS Properties   │
│ _states         │   │ Data Transform  │     │                        │
└─────────────────┘   └─────────────────┘     └────────────────────────┘
                      
```
## 🔧 System Requirements

### Docker Execution (Recommended)
- **Docker** 20.10+  
- **Docker Compose** 1.29+  
- **Minimum 2GB RAM** for containers

> Note: This repository distributes runtime images and is intended to be run with Docker. Local Python execution instructions have been removed.

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
| **robot-data** | 🤖 Robot Simulator | - | Publishes realistic robot joint state data to Kafka topic `MPCM.joint_tool_states` |
| **kafka-ui** | 🧭 Kafka UI | 8090 | Web UI for Kafka clusters (optional) |


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

# Check service health
docker-compose ps

# View logs from specific services
docker-compose logs -f databridge-configurator
docker-compose logs -f robot-data-generator
```

### ✅ Verification

1. Verify robot data in Kafka UI
   - Open Kafka UI: http://localhost:8090
   - Go to Topics → search for `MPCM.joint_tool_states`.
   - Open the topic and view/consume messages. You should see JSON messages being published by the robot simulator.

2. Verify live data in BaSyx AAS Web UI
   - Open BaSyx AAS Web UI: http://localhost:3000
   - Locate the **Robot** AAS and open **Operational Data** submodel/properties view.
   - Enable synchronization for the Robot AAS (toggle top right button labelled "Auto sync").
   - Observe the AAS properties update in real time as messages arrive on the Kafka topic.


## ⚙️ Configuration

### Environment Variables

Configure your environment:

```bash
# Edit configuration
nano .env
```

The `.env` file contains the BaSyx server configurations and submodel identifiers:

```properties
BASE_URL=http://aas-env:8081
DATABRIDGE_ID=aHR0cHM6Ly93d3cuYWltZW4uZXMvYWFzL3h4bHBpbG90ZmFjdG9yeS9kYXRhYnJpZGdl
DATABRIDGE_ID_AID=aHR0cHM6Ly93d3cuYWltZW4uZXMvc20vYWlkL3h4bGlpaWxvdGZhY3RvcnkvZGF0YWJyaWRnZQ==
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


## 🙏 Acknowledgments

This project is developed as part of the **CONVERGING** project, which aims to develop, deploy, validate, and promote smart and reconfigurable production systems including multiple autonomous agents (collaborative robots, AGVs, humans) that are able to act in diverse production environments.

🌐 **Project Website**: [https://www.converging-project.eu/](https://www.converging-project.eu/)

🇪🇺 **Funding**: The CONVERGING project is co-funded by the European Union's Horizon Europe Research & Innovation Programme under Grant N° 101058521.

We acknowledge the support and contributions of all CONVERGING project partners and collaborators who have made this work possible.
