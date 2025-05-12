# Reppo Solver Node 🌟
                   

![Reppo Solver Node](https://img.shields.io/badge/Version-0.1.0-blue.svg) ![License](https://img.shields.io/badge/License-MIT-green.svg)

The **Reppo Solver Node** is a decentralized application designed to participate in the **Reppo.Exchange**, a blockchain-based data marketplace that facilitates the creation, validation, and exchange of high-quality datasets. The Solver Node listens for Requests for Data (RFDs), generates or sources the requested datasets, uploads them to IPFS, verifies NFT ownership for access control, and submits solutions to the Reppo Exchange smart contract.

This README provides an overview of the Reppo Solver Node, its architecture, setup instructions, and usage guidelines.

---

## Table of Contents 📋

- [Overview](#overview)
- [Architecture](#architecture)
  - [Components](#components)
  - [Workflow](#workflow)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Running in Test Mode](#running-in-test-mode)
  - [Running in Production Mode](#running-in-production-mode)
- [Example RFD](#example-rfd)
- [Dependencies](#dependencies)
- [Contributing](#contributing)
- [License](#license)

---

## Overview ✨

**Reppo.Exchange** is a decentralized data exchange platform that enables data buyers (Requestors) to post **Requests for Data (RFDs)** specifying their dataset requirements. Solver Nodes, like this one, respond to these RFDs by generating or sourcing the requested data, validating it, and submitting it to the platform for rewards. The platform leverages blockchain technology (Ethereum), IPFS for decentralized storage, and NFTs for access control to ensure transparency, security, and incentivization.

The Reppo Solver Node is a key participant in this ecosystem, performing the following tasks:

1. **Listening for RFDs**: Monitors the Reppo Exchange smart contract for new RFD events.
2. **Generating Datasets**: Uses a data generation service (e.g., Two Ligma server) to create synthetic or real datasets that meet RFD specifications.
3. **Uploading to IPFS**: Stores datasets on IPFS via Pinata for decentralized, persistent storage.
4. **Verifying NFT Ownership**: Ensures the node operator owns a Reppo Node NFT, which is required to participate in the network.
5. **Submitting Solutions**: Submits the IPFS URI of the dataset to the Reppo Exchange smart contract for validation and reward distribution.

The Solver Node is designed to be modular, extensible, and easy to integrate with various data sources, such as Vana (Data DAOs), self-hosted LLMs, OpenGradient (synthetic data), or enterprise datasets.

---

## Architecture 🏗️

The Reppo Solver Node is built as a Python application with a modular architecture, separating concerns into distinct components. This design ensures maintainability, scalability, and flexibility for integrating with different data sources and blockchain networks.

### Components

The Solver Node consists of the following key components, each implemented in a dedicated Python module:

1. **RFDListener (`rfdListener.py`)**
   - **Purpose**: Listens for `RFDPosted` events emitted by the Reppo Exchange smart contract.
   - **Functionality**: Uses `web3.py` to connect to an Ethereum node (via `WEB3_RPC_URL`) and polls for new RFD events. When an RFD is detected, it parses the event data into a dictionary and passes it to a callback function for processing.
   - **Dependencies**: `web3.py`, `python-dotenv`.

2. **DataSolver (`dataSolver.py`)**
   - **Purpose**: Generates or sources datasets that fulfill RFD requirements.
   - **Functionality**: Communicates with a **Two Ligma server** (a data generation service) via HTTP requests to produce synthetic datasets. Saves the generated dataset as a JSON file in the `data/` directory.
   - **Dependencies**: `requests`, `python-dotenv`.

3. **IPFSUploader (`ipfsUploader.py`)**
   - **Purpose**: Uploads datasets to IPFS for decentralized storage.
   - **Functionality**: Uses the Pinata API to pin files to IPFS, returning an `ipfs://<CID>` URI. Handles file uploads securely with API key authentication.
   - **Dependencies**: `requests`, `python-dotenv`.

4. **NFTAuthorizer (`nftAuthorizer.py`)**
   - **Purpose**: Verifies that the node operator’s wallet owns a Reppo Node NFT, which is required to submit solutions.
   - **Functionality**: Queries an ERC-721 NFT contract using `web3.py` to check the `balanceOf` the wallet address. Supports block-specific queries for historical ownership verification.
   - **Dependencies**: `web3.py`, `python-dotenv`.

5. **SolutionSubmitter (`submitSolution.py`)**
   - **Purpose**: Submits the dataset’s IPFS URI to the Reppo Exchange smart contract.
   - **Functionality**: Builds, signs, and sends a transaction to the `submitSolution` function of the smart contract, including the RFD ID and IPFS URI. Returns the transaction hash upon success.
   - **Dependencies**: `web3.py`, `python-dotenv`.

6. **SolverNode (`solverNode.py`)**
   - **Purpose**: Orchestrates the entire workflow by integrating all components.
   - **Functionality**: Initializes the other components, coordinates RFD processing, and supports test and mock modes for development. In production mode, it runs the listener and processes RFDs as they arrive.
   - **Dependencies**: All other modules.

7. **Main CLI (`main.py`)**
   - **Purpose**: Provides a command-line interface (CLI) to start the Solver Node.
   - **Functionality**: Uses the `click` library to offer options like `--test` (for testing with a sample RFD) and `--mock` (for offline testing without blockchain interactions).
   - **Dependencies**: `click`, `python-dotenv`.

### Workflow

The Solver Node follows this workflow to process an RFD:

1. **RFD Detection**:
   - The `RFDListener` detects a new `RFDPosted` event on the Reppo Exchange smart contract.
   - The RFD details (ID, name, description, schema) are extracted and passed to the `SolverNode`.

2. **NFT Verification**:
   - The `NFTAuthorizer` checks if the wallet address (`WALLET_ADDRESS`) owns a Reppo Node NFT.
   - If no NFT is found, the RFD is skipped.

3. **Dataset Generation**:
   - The `DataSolver` sends the RFD to the Two Ligma server, which generates a synthetic dataset.
   - The dataset is saved locally as `data/rfd_<rfd_id>_solution.json`.

4. **IPFS Upload**:
   - The `IPFSUploader` uploads the dataset to IPFS via Pinata, returning an `ipfs://<CID>` URI.

5. **Solution Submission**:
   - The `SolutionSubmitter` builds a transaction to call the `submitSolution` function on the Reppo Exchange smart contract, passing the RFD ID and IPFS URI.
   - The transaction is signed with the private key (`PRIVATE_KEY`) and sent to the Ethereum network.
   - The transaction hash is logged for tracking.

6. **Completion**:
   - If successful, the dataset is available on the Reppo Exchange for validation and reward distribution.
   - The Solver Node continues listening for new RFDs.

---

## Prerequisites ✅

To run the Reppo Solver Node, ensure you have the following:

- **Python**: Version 3.8 or higher.
- **Ethereum Node Access**: A connection to an Ethereum node (e.g., Infura, Alchemy) via an RPC URL.
- **Pinata Account**: API keys for IPFS pinning (`PINATA_API_KEY` and `PINATA_SECRET_API_KEY`).
- **Two Ligma Server**: A running instance of the Two Ligma server for dataset generation (default URL: `http://localhost:5000/api/agent/run`).
- **Reppo Node NFT**: Ownership of a Reppo Node NFT in the wallet used for submission.
- **Environment Variables**: A `.env` file with required configurations (see [Configuration](#configuration)).
- **Dependencies**: Python packages listed in `requirements.txt` (see [Dependencies](#dependencies)).

---

## Installation 💾

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Reppo-Labs/solver-node
   cd reppo-solver-node
   ```

2. **Set Up a Virtual Environment** (recommended):
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Create the Output Directory**:
   ```bash
   mkdir data
   ```

5. **Configure Environment Variables**:
   Create a `.env` file in the project root (see [Configuration](#configuration)).

---

## Configuration ⚙️

The Solver Node relies on environment variables defined in a `.env` file. Below is an example `.env` file:

```env
# Ethereum configuration
WEB3_RPC_URL=https://mainnet.infura.io/v3/your-infura-key
CHAIN_ID=1
WALLET_ADDRESS=0xYourWalletAddress
PRIVATE_KEY=your-private-key

# Reppo Exchange smart contract
EXCHANGE_CONTRACT_ADDRESS=0xExchangeContractAddress
EXCHANGE_CONTRACT_ABI_PATH=./abis/exchange_abi.json

# Reppo Node NFT contract
NFT_CONTRACT_ADDRESS=0xNFTContractAddress
NFT_CONTRACT_ABI_PATH=./abis/nft_abi.json

# Pinata IPFS configuration
PINATA_API_KEY=your-pinata-api-key
PINATA_SECRET_API_KEY=your-pinata-secret-api-key

# Two Ligma server (optional, defaults to localhost)
DATANODE_URL=http://localhost:5000/api/agent/run
```

**Security Notes**:
- Never commit your `.env` file to version control.
- Keep your `PRIVATE_KEY` and Pinata API keys secure.
- Ensure the `data/` directory has write permissions for the user running the node.

---

## Usage 🖥️

The Solver Node can be run in different modes depending on your needs:

### Running in Test Mode 🧪

Test mode allows you to process a sample RFD without connecting to the blockchain or Two Ligma server.

1. **Prepare a Sample RFD**:
   Create a `sample_rfd.json` file in the project root. For example:
   ```json
   {
       "rfd_id": "sf_weather_may_aug_001",
       "name": "Synthetic Weather Data for San Francisco (May to August)",
       "description": "A synthetic dataset containing daily weather information for San Francisco from May to August, including temperature, humidity, and precipitation.",
       "schema": {
           "type": "object",
           "properties": {
               "date": { "type": "string", "format": "date" },
               "temperature": { "type": "number", "description": "Average daily temperature in degrees Fahrenheit" },
               "humidity": { "type": "number", "description": "Average daily humidity percentage" },
               "precipitation": { "type": "number", "description": "Daily precipitation in inches" }
           },
           "required": ["date", "temperature", "humidity", "precipitation"]
       }
   }
   ```

2. **Run in Test Mode**:
   ```bash
   python main.py start --test
   ```
   - This processes the `sample_rfd.json` file and generates a dataset in the `data/` directory.
   - Blockchain interactions are skipped unless `--mock` is also used.

3. **Run in Mock Mode** (optional):
   ```bash
   python main.py start --test --mock
   ```
   - Mocks blockchain and IPFS interactions, returning dummy values (e.g., `ipfs://mockCID`, `0xMockTransactionHash`).

### Running in Production Mode 🚀

Production mode runs the Solver Node as a listener for live RFDs on the Reppo Exchange.

1. **Ensure Configuration**:
   Verify that your `.env` file is correctly set up and that the Two Ligma server is running.

2. **Start the Node**:
   ```bash
   python main.py start
   ```
   - The node connects to the Ethereum network, listens for `RFDPosted` events, and processes RFDs as they arrive.
   - Datasets are generated, uploaded to IPFS, and submitted to the smart contract.

3. **Monitor Logs**:
   - The node logs progress to the console, including RFD detection, dataset generation, IPFS uploads, and transaction hashes.
   - Check the `data/` directory for generated datasets.

---

## Example RFD 📊

Below is an example RFD for synthetic weather data, which the Solver Node can process:

```json
{
    "rfd_id": "sf_weather_may_aug_001",
    "name": "Synthetic Weather Data for San Francisco (May to August)",
    "description": "A synthetic dataset containing daily weather information for San Francisco from May to August, including temperature, humidity, and precipitation.",
    "schema": {
        "type": "object",
        "properties": {
            "date": {
                "type": "string",
                "format": "date"
            },
            "temperature": {
                "type": "number",
                "description": "Average daily temperature in degrees Fahrenheit"
            },
            "humidity": {
                "type": "number",
                "description": "Average daily humidity percentage"
            },
            "precipitation": {
                "type": "number",
                "description": "Daily precipitation in inches"
            }
        },
        "required": ["date", "temperature", "humidity", "precipitation"]
    }
}
```

When processed, the Solver Node:
- Generates a dataset matching the schema (e.g., daily weather data for May–August).
- Saves it as `data/rfd_sf_weather_may_aug_001_solution.json`.
- Uploads it to IPFS, obtaining an `ipfs://<CID>` URI.
- Submits the URI to the Reppo Exchange smart contract if the wallet owns a Reppo Node NFT.

---

## Dependencies 📦

The Reppo Solver Node relies on the following Python packages, listed in `requirements.txt`:

```text
# Blockchain interactions
web3>=6.10.0
eth-account>=0.9.0
eth-typing>=3.4.0
eth-utils>=2.2.0

# HTTP requests
requests>=2.31.0

# Environment variables
python-dotenv>=1.0.0

# JSON processing
json5>=0.9.14

# CLI interface
click>=8.1.7

# Testing
pytest>=7.4.0
pytest-mock>=3.11.1
```

Install them using:
```bash
pip install -r requirements.txt
```

---

## Contributing 🤝

We welcome contributions to improve the Reppo Solver Node! To contribute:

1. **Fork the Repository**:
   ```bash
   git clone https://github.com/your-repo/reppo-solver-node.git
   ```

2. **Create a Feature Branch**:
   ```bash
   git checkout -b feature/your-feature
   ```

3. **Make Changes**:
   - Follow PEP 8 for Python code style.
   - Add tests for new functionality in the `tests/` directory.
   - Update documentation if necessary.

4. **Submit a Pull Request**:
   - Push your branch to your fork and create a pull request against the main repository.
   - Describe your changes clearly in the pull request description.

5. **Run Tests**:
   ```bash
   pytest
   ```

Please report issues or feature requests via the GitHub issue tracker.

---

## License 📜

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

This README provides a comprehensive guide to understanding, setting up, and running the Reppo Solver Node. For further questions, please open an issue on the GitHub repository or contact the maintainers.
