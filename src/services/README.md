# index.ts

The `index.ts` file within the `src/services` directory of the `ferrumnet/multiswap-generator-node` repository serves as a central point for exporting modules. This file essentially organizes and makes accessible the defined modules in the project for periodic tasks.

# axios.service.ts

The file `axios.service.ts` contains three primary functions related to HTTP operations using the `axios` library, each designed to interact with the MultiSwap backend. Below are detailed descriptions of each function:

### 1\. getTransactions

This asynchronous function retrieves a list of pending swap transactions from the MultiSwap backend. It constructs a URL based on whether the environment is local or AWS, and appends query parameters specifying transaction status, node type, and the public key from the environment. The function includes authorization headers using a bearer token generated by `createAuthTokenForMultiswapBackend()`. If successful, it returns the transactions array from the response body. In case of failure, it logs the error and returns null.

typescriptCopy code

`export let getTransactions = async function () {
  try {
    let baseUrl = (global as any).AWS_ENVIRONMENT.BASE_URL_MULTISWAP_BACKEND;
    if (process.env.ENVIRONMENT == "local") {
      baseUrl = "http://localhost:8080";
    }
    let config = {
      headers: {
        Authorization: BEARER + createAuthTokenForMultiswapBackend(),
      },
    };
    let url = `${baseUrl}/api/v1/transactions/list?status=swapPending&limit=20&nodeType=generator&address=${
      (global as any).AWS_ENVIRONMENT.PUBLIC_KEY
    }`;
    let res = await axios.get(url, config);
    return res.data.body.transactions;
  } catch (error) {
    console.log(error);
    return null;
  }
};`

### 2\. updateTransaction

This asynchronous function is used to update a transaction's status based on the transaction hash (`txHash`). It sends a PUT request to the MultiSwap backend with the transaction hash and additional body data. It similarly configures the base URL and headers as in `getTransactions`. The function returns the response from the backend.

typescriptCopy code

`export const updateTransaction = async (txHash: string, body: any) => {
  let baseUrl = (global as any).AWS_ENVIRONMENT.BASE_URL_MULTISWAP_BACKEND;
  if (process.env.ENVIRONMENT == "local") {
    baseUrl = "http://localhost:8080";
  }
  let config = {
    headers: {
      Authorization: BEARER + createAuthTokenForMultiswapBackend(),
    },
  };
  return axios.put(
    `${baseUrl}/api/v1/transactions/update/from/generator/${txHash}?address=${
      (global as any).AWS_ENVIRONMENT.PUBLIC_KEY
    }`,
    body,
    config
  );
};`

### 3\. getRpcNodes

This asynchronous function fetches a list of RPC nodes from the MultiSwap backend. The function constructs a GET URL that includes the public key and specifies that the node type is a generator and pagination is disabled. It uses the same URL and header configuration as the other functions. The function returns the data array from the response body or null in the event of an error.

typescriptCopy code

`export let getRpcNodes = async function () {
  try {
    let baseUrl = (global as any).AWS_ENVIRONMENT.BASE_URL_MULTISWAP_BACKEND;
    if (process.env.ENVIRONMENT == "local") {
      baseUrl = "http://localhost:8080";
    }
    let config = {
      headers: {
        Authorization: BEARER + createAuthTokenForMultiswapBackend(),
      },
    };
    let url = `${baseUrl}/api/v1/rpcNodes/list?address=${
      (global as any).AWS_ENVIRONMENT.PUBLIC_KEY
    }&nodeType=generator&isPagination=false`;
    let res = await axios.get(url, config);
    return res.data.body.data;
  } catch (error) {
    console.log(error);
    return null;
  }
};`

These functions are designed to ensure secure and authorized communication with the backend services, leveraging environment-based configurations to suit different deployment scenarios.

# cosmWasm.service.ts

Here's a detailed documentation for each function found in the cosmWasm.service.ts file of the GitHub repository:

### `getTransactionReceipt(txId: string, rpcURL: string, threshold: number = 0, tries: number = 0): Promise<TransactionReceipt>`

This asynchronous function retrieves a transaction receipt given a transaction ID and an RPC URL. It checks the transaction status and, if the number of attempts is less than the threshold, it recursively calls itself to retry fetching the transaction receipt. It returns a `TransactionReceipt` object which includes the transaction status.

### `signedTransaction(job: any, decodedData: any, transaction: any): Promise<any>`

This asynchronous function handles the process of creating a signed transaction. It calculates the destination amount, prepares transaction data including hash and signatures, and then calls `createSignedPayment` to sign the transaction data. The function catches and logs any errors during the process.

### `createSignedPayment(chainId: string, address: string, amount: string, token: string, salt: string, job: any): Promise<any>`

This function creates a payment signature based on blockchain transaction details. It constructs a withdrawal hash, retrieves a private key, and uses an Ethereum wallet to sign the hash. The function returns an object containing the signature and hash of the payment.

### `produceSignatureWithdrawHash(chainId: string, token: string, payee: string, amount: string, salt: string): any`

This function produces a hash for signature withdrawal based on several transaction parameters. It generates a JSON string that includes chain ID, payee, token, amount, and a salt, and returns an object containing the hash and an empty array of signatures.

### `getLogsFromTransactionReceipt(job: any)`

This function parses and processes logs from a transaction receipt to extract transaction-related data such as tokens, amounts, addresses, and status. It filters and retrieves values from the logs and constructs a decoded data object.

### `filterLogsAndGetValue(logs: any, key: string)`

This helper function filters logs to find and return a value associated with a specified key. It iterates through the events and attributes in the logs to match the key.

### `getDestinationAmount(data: any): Promise<any>`

This function returns the `swapBridgeAmount` from the provided data, typically used to determine the amount to be used in a blockchain transaction.

These functions together assist in managing and interacting with transactions on a blockchain via CosmWasm, particularly focusing on handling transaction receipts, signing transactions, and extracting relevant data from transaction logs.

# rpcNode.service.ts

This TypeScript file from the `multiswap-generator-node` repository provides services related to RPC (Remote Procedure Call) nodes. Below is a detailed breakdown of each function within the file:

#### 1\. `saveRpcNodes(data: [RpcNode])`

Purpose: This asynchronous function saves an array of `RpcNode` objects. Parameters:

-   `data: [RpcNode]`: An array of `RpcNode` objects to be saved. Implementation:
-   Calls `setRpcNodesData(data)` to save the RPC node data.

#### 2\. `getRpcNodeByChainId(chainId: string): RpcNode`

Purpose: Retrieves an RPC node based on the given chain ID. Parameters:

-   `chainId: string`: The blockchain's chain ID for which the RPC node information is needed. Implementation:
-   Retrieves all RPC nodes data using `getRpcNodesData()`.
-   Searches through the data to find an RPC node matching the provided chain ID.
-   Returns the found node if any; otherwise, returns a default object with the provided chain ID and an empty URL. Error Handling:
-   Catches and logs any errors that occur during the search process.

Each function is designed to manage and retrieve information about RPC nodes, which are critical for blockchain interactions. This code is structured to ensure efficient data retrieval and handling, allowing for seamless integration within larger blockchain services.

# signature.service.ts

Here's a detailed documentation of each function found in the `signature.service.ts` file from the specified GitHub repository:

### 1\. `getDataForSignature(job: any, decodedData: any, transaction: any): Promise<any>`

This function constructs and returns transaction data based on various inputs including job data, decoded data, and transaction details. It integrates data related to blockchain transactions, including addresses, chain IDs, token information, and amounts. The function also manages signature data using other functions to validate and process withdrawal data.

### 2\. `getValidWithdrawalData(data: any, decodedData: any): Promise<any>`

Validates the withdrawal data against the latest hash generated from the transaction details. It returns relevant transaction data if the hashed data matches and the settled amount is valid. Otherwise, it returns null.

### 3\. `isValidSettledAmount(slippage: number, sourceChainId: string, destinationChainId: string, destinationAmountIn: any, settledAmount: any): Promise<boolean>`

Calculates and validates whether the settled amount is within acceptable limits based on the slippage percentage. It ensures the transaction meets the required conditions of amount slippage between different blockchain networks.

### 4\. `createSignedPayment(...)`

Generates a signed payment based on the transaction type. It supports both Foundary and OneInch transaction types by generating appropriate hashes and then signing them using the ECDSA signing algorithm. The function returns the signature and hash of the transaction.

### 5\. `produceFoundaryHash(...)`

Constructs and hashes a transaction payload specific to Foundary transactions. It organizes the data into a specific format and computes the hash required for signing.

### 6\. `produceOneInchHash(...)`

Constructs and hashes a transaction payload specific to OneInch transactions. It prepares and hashes the data, similar to the Foundary transaction but tailored for OneInch swaps.

### 7\. `domainSeparator(web3: Web3, chainId: string, contractAddress: string)`

Generates a domain separator for EIP712 typed data signing, crucial for verifying the integrity and origin of the signed data.

### 8\. `fixSig(sig: any)`

Adjusts the signature 'v' value to correct for potential mismatches in ECDSA recovery parameters.

### 9\. `getDataForSalt(txData: any, decodedData: any): string`

Generates a salt string based on transaction hash and decoded data, used for further cryptographic processes.

### 10\. `getDecodedLogsDataIntoString(decodedData: any): string`

Converts multiple transaction-related data points into a single string, mainly for logging or further data handling.

These functions collectively handle the preparation, validation, and signing of blockchain transaction data, ensuring integrity and compatibility with different blockchain protocols.

# transaction.service.ts

Here's the documentation for each function found in the `transaction.service.ts` file:

### Function: `fetchChainDataFromNetwork`

This function is responsible for fetching transaction-related data across different networks. It accepts a transaction object (`tx`) as an argument. If the transaction exists, it extracts necessary details such as source and destination network information, RPC URLs, and transaction-specific data (e.g., asset types, amounts, and transaction IDs). Depending on the network type (EVM or non-EVM), it calls appropriate services to get the transaction receipt. If the transaction is successfully retrieved and confirmed, it triggers the creation of a signature. If not, it logs a failure and updates the transaction status.

### Function: `createSignature`

This function generates a signature for a job that encapsulates transaction data. It determines whether the source or destination is non-EVM and fetches the transaction logs accordingly. It then proceeds to generate a signed transaction using the fetched data. Once a signature is created, the function updates the transaction with this new data.

### Function: `updateTransaction`

This function updates a given transaction with new data such as signed data and transaction details. It also handles the removal of the transaction hash from a local list after updating, to maintain the accuracy and cleanliness of local data records. If an error occurs during the update process, it logs the error.

The functions interact with various services (`web3Service`, `cosmWasmService`, and `axiosService`) to manage transaction data across blockchain networks, demonstrating a modular approach to handling blockchain interactions.

# web3.service.ts

Here are detailed documentations for each function found in the `web3.service.ts` file from the repository:

### `getTransactionReceipt(txId: string, chainId: string, threshold: number, tries = 0): Promise<TransactionReceipt>`

This asynchronous function retrieves a transaction receipt given a transaction ID and a chain ID. It makes recursive calls up to a specified threshold if the transaction receipt is not initially retrieved or if it lacks a status. It utilizes a delay between retries.

### `getTransactionByHash(txHash: string, chainId: string): Promise<Transaction>`

Given a transaction hash and a chain ID, this asynchronous function fetches the transaction details using Web3.

### `signedTransaction(job: any, decodedData: any, transaction: any): Promise<any>`

This function handles the process of signing a transaction. It constructs transaction data, calculates a hash (salt), and creates a signed payment transaction. The function expects a job object, decoded data, and the transaction data, returning a structured object with the transaction details and signatures.

### `getLogsFromTransactionReceipt(job: any)`

Extracts and decodes logs from a transaction receipt. This function searches through log entries for specific event topics related to swaps and decodes them if found.

### `findSwapEvent(topics: any[], job: any)`

A helper function that searches for a swap event hash in a list of topics. It returns the index of the swap event hash if found or `undefined` otherwise.

### `getFundManagerAddress(chainId: string)`

Retrieves the address of the fund manager for a given chain ID by searching through a predefined list of networks.

### `getFiberRouterAddress(chainId: string)`

Retrieves the address of the Fiber Router for a given chain ID by searching through a predefined list of networks.

### `getFoundaryTokenAddress(chainId: string)`

Fetches the address of the Foundary Token for a specific chain ID from a list of known networks.

### `checkValidTransactionAndReturnReceipt(txId: string, chainId: string, receipt: TransactionReceipt): Promise<any>`

Checks if a given transaction is valid by comparing its destination address to the Fiber Router address for a given chain ID. If valid, it returns the receipt; otherwise, it returns null.

### `delay()`

A utility function that introduces a delay, primarily used for throttling requests or retries in functions.
