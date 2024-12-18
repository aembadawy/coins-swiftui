# Networking in Swift with Completion Handlers

## URL Session
1- ```URLSession.shared.datatask ``` creates a **susbended** task that retrieves the contents of a url request and then executes a call back upon completion.
2- Completion handlers are a call back from the API that begins whenever the call request concludes.
3- This code executes asynchronously or non-sequentially as we don't know the execution time of the API request, and therefore must be executed on a `background thread`.
4- Data returns as JSON (JavaScript Object Notation) is a lightweight, text-based format for storing and exchanging data. 
It uses key-value pairs and is easy for humans to read and machines to parse.

Once we received the data we have 2 options to decode the JSON object 
1. **`JSONSerializer`**: This is a general term that usually refers to the process of converting objects to and from JSON. In Swift, this is often done using `JSONSerialization`, which provides methods for converting JSON data into Swift objects and vice versa. It’s a more manual process and requires you to handle the JSON data as `Any` or `Dictionary` types. In summary, `JSONSerialization` is more manual

2. **`JSONDecoder().decode`**: This is a method provided by Swift’s `JSONDecoder` class that allows for the automatic decoding of JSON data into strongly-typed Swift objects. It uses Swift’s `Codable` protocol, which makes it easier to map JSON data directly to your custom data structures with type safety and less manual parsing code. In summary, `JSONDecoder().decode` is more streamlined and type-safe.

## Threading
Updating the user interface must happen on the main thread as it's the `Main path of execution` and gets the most computing power for the app to run smoothly. Not only *API Calls* but also *complex operation*, and *heavy calculations* must execute on a background thread to maintain smooth user experience.

## @escaping 
The escaping keyword indicates that we need the property which we are calling this completion handler on to escape this block of code. 
Mainly used to mark a closure (a block of code that can be passed around and executed later) that is allowed to outlive the scope in which it was created.

## The Result Type

The `Result` type in Swift is a powerful and flexible way to handle operations that can either succeed with a value or fail with an error. 

#### Definition

The `Result` type is an enum with two cases:

1. **`success`**: Represents a successful operation and holds an associated value of the result type.
2. **`failure`**: Represents a failed operation and holds an associated error value.


#### Syntax

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

- `Success`: The type of the value that is returned if the operation succeeds.
- `Failure`: The type of the error that is returned if the operation fails. It must conform to the `Error` protocol.

#### Key Points

- **Type Safety**: The `Result` type enforces type safety by requiring explicit handling of both success and failure cases.
- **Clarity**: It makes the code clearer by showing that the function can return either a success or failure case, and handling these cases becomes more straightforward.
- **Error Handling**: The `failure` case provides a way to carry an error object that conforms to the `Error` protocol, allowing for detailed error reporting and handling.

## The do-try-catch model

In Swift, `try`, `do`, and `catch` are used to handle errors in a controlled manner when working with functions that throw errors. 

- **`try`**: Indicates that a function or method can throw an error and you need to handle it.
- **`do`**: Starts a block of code where you can call throwing functions. Can be followed by *one or more* catch statements.
- **`catch`**: Handles errors thrown within the `do` block and provides ways to handle different error types.

## Retain Cycles

When you create a reference to an object in another object, in swift, this automatically results in strong reference.
Strong references mean that the 2 objects will keep eachother alive in memory even when one is destroyed. 
This leads to memory leaks, waisted memory, unexpected behavior and poor performance.

### Example 

```
private let urlString = "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=20&page=1&sparkline=false&price_change_percentage=24h&locale=en"
    
    func fetchCoinsWithResult(completion: @escaping(Result<[Coin], APError>) -> Void) {
        guard let url = URL(string: urlString) else { return }
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                print("Debug: Error \(error.localizedDescription)")
                completion(.failure(.unknownError(description: error.localizedDescription)))
            }
            
            guard let httpResponse = response else {
                completion(.failure(.requestFailed(description: "Request Failed")))
                return
            }

            guard let data = data else {
                completion(.failure(.invalidData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode([Coin].self, from: data)
                completion(.success(response))
            } catch { //we get access to error inside the catch block by default
                print("Debug: Failed to decode object \(error)")
                completion(.failure(.jsonParsingFailure))
            }
        }.resume()
```

## Caching

Caching allows us to store information that doesn’t change frequently, reducing the need to fetch data from the network on every user interaction. 

### Why Use Caching?
- **Cost-Effective**: Retrieving cached data is less expensive than making network calls.
- **Reduced Network Calls**: Caching minimizes the number of requests sent to the server, which can improve performance and decrease load on the network.
- **Speed**: Accessing cached data is generally faster than retrieving it from the network.
- **Battery Efficiency**: Fewer network calls mean less battery consumption, enhancing the user experience.

### What Are the Trade-offs?
- **Memory Usage**: Caching takes up space on the device. We need to implement caching policies to manage this effectively and avoid excessive memory use.
- **Stale Data**: Cached information can become outdated. To address this, we may need to implement logic to refresh data after a certain period or use techniques like Least Recently Used (LRU) to manage cache entries effectively.
