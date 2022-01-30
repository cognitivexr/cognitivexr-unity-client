# cognitivexr-unity-client
Unity Client for the CognitiveXR Client

## Installation

Tested with Unity 2019.4.30f1

### Dependencies

1. Add NuGet for Unity by following the instructions https://github.com/GlitchEnzo/NuGetForUnity
2. Install MQTTnet 3.1.1 via the NuGet package manager
3. Add the cognitivexr client via the Unity package manager with the option ```add package from git URL...``` and add 'https://github.com/cognitivexr/cognitivexr-unity-client'. This downloads the cognitivexr client from github.
4. Install MRTK by following the instructions on https://docs.microsoft.com/en-us/windows/mixed-reality/mrtk-unity/?view=mrtkunity-2021-05

## CPOP Client

CPOP uses MQTT for commmunication and returns the following informations:
```
CpopData:
    float Timestamp;
    string Type;
    Coordinates Position;
    List<Coordinates> Shape;
```

### How to connect to the server
```
CpopSubscriber cpopSubscriber;
string CpopServerAddress = "localhost";

cpopSubscriber = new CpopSubscriber(new CpopServerOptions{ Server = CpopServerAddress});
        cpopSubscriber.Subscribe();
```
replace `localhost` with the IP adress of the cpop server.

### How to get data from the CPOP client

```
private void Update()
{
    if (cpopSubscriber.Queue.TryDequeue(out CpopData cpopData))
    {
        // your implementation
    }
}
```
Sample: https://github.com/cognitivexr/unity-demo-app CPOPExample.unity

## CogStream Client

### Connect to Mediator
1. Connect to the mediator

```
//create a mediator client
mediatorClient = new MediatorClient("ws://192.168.1.1");
mediatorClient.Open();
```
2. Get available engines

```
// returns a list of available engines from the mediator 
List<Engine> engines = await mediatorClient.GetEngines();
```

3. Starting an engine
```
// returns the address of the engine after launch
string address = await mediatorClient.StartEngine(engine);
```

### Interact with Engine
After telling the mediator to launch an engine a connection to it can be established.
An engine consists out of two parts. A sender component and a receiver componen


### Connect to Engine

Create a StreamSpec object to configure the connection
```
StreamSpec streamSpec = new StreamSpec
{
    engineAddress = engineAddress, // address of the engine that the mediator returned
    attributes = new Attributes()  // additional configurations
};
```

```
// create a send channel
JpgSendChannel sendChannel = new JpgSendChannel(resolution.width, resolution.height);
// create receive channel
FaceReceiveChannel receiveChannel = new FaceReceiveChannel()

// start engine
EngineClient engineClient = new EngineClient(streamSpec, sendChannel, receiveChannel);
engineClient.Open();
```

### Sender Channel
Implements the serialization of sending data to the engine. For example `JpgSendChannel` encodes a jpeg and sends it to the engine.

```
JpgSendChannel jpgSendChannel = engineClient.GetSendChannel<JpgSendChannel>();
uint? frameId = jpgSendChannel?.Send(frame);
```

### Receive Channel
Waits for new messages from the engine and implements the parsing of them.
For example `DebugReceiveChannel` 

#### How to use the reveive channel:
```
// get receive channel from engine
var receiveChannel = engineClient.GetReceiveChannel<FaceReceiveChannel>();

// gets engine results 
List<FaceEngineResult> faceEngineResults = await receiveChannel.Next<FaceEngineResult>();
```
Example: https://github.com/cognitivexr/unity-demo-app CogStreamExample.unity
