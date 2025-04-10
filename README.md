# smart-sustainability

Creating a Python-based application for monitoring and optimizing energy consumption in smart homes involves integrating IoT data, processing it, and potentially providing actionable insights for smart home users. Below is a simplified template of how such an application might look. This example uses a combination of Python libraries like `paho-mqtt` for IoT data communication, `pandas` for data manipulation, and `matplotlib` for visualization. Additionally, it includes error handling and comments for clarity.

```python
import paho.mqtt.client as mqtt
import pandas as pd
import matplotlib.pyplot as plt
import json
import sys

class SmartSustainabilityApp:
    def __init__(self, broker_address="localhost", topic="energy/data"):
        self.broker_address = broker_address
        self.topic = topic
        self.energy_data = pd.DataFrame(columns=['timestamp', 'device_id', 'energy_consumption'])

    def on_connect(self, client, userdata, flags, rc):
        """Handles connection to the MQTT broker."""
        if rc == 0:
            print("Connected to broker")
            client.subscribe(self.topic)
        else:
            print(f"Failed to connect, return code {rc}")

    def on_message(self, client, userdata, message):
        """Handles incoming messages from the MQTT broker."""
        try:
            msg = json.loads(message.payload.decode())
            new_data = {
                'timestamp': pd.to_datetime(msg['timestamp']),
                'device_id': msg['device_id'],
                'energy_consumption': msg['energy_consumption']
            }
            self.energy_data = self.energy_data.append(new_data, ignore_index=True)
            print(f"Received data: {new_data}")
        except json.JSONDecodeError as e:
            print(f"Error decoding JSON: {e}")
        except KeyError as e:
            print(f"Missing expected key in data: {e}")

    def analyze_energy_data(self):
        """Analyzes the collected energy data."""
        try:
            summary = self.energy_data.groupby('device_id')['energy_consumption'].sum().sort_values(ascending=False)
            print("Energy consumption by device:")
            print(summary)
            self.plot_energy_consumption(summary)
        except Exception as e:
            print(f"An error occurred during data analysis: {e}")

    def plot_energy_consumption(self, summary):
        """Plots the energy consumption data."""
        try:
            summary.plot(kind='bar')
            plt.title('Energy Consumption by Device')
            plt.xlabel('Device ID')
            plt.ylabel('Energy Consumption (kWh)')
            plt.show()
        except Exception as e:
            print(f"An error occurred while plotting data: {e}")

    def run(self):
        """Runs the MQTT client to collect and process data."""
        client = mqtt.Client("SmartSustainabilityApp")
        client.on_connect = self.on_connect
        client.on_message = self.on_message

        try:
            client.connect(self.broker_address)
            client.loop_start()
            
            # Keep the script running to collect data indefinitely
            while True:
                command = input("Type 'analyze' to view energy analysis or 'exit' to quit: ").strip().lower()
                if command == "analyze":
                    self.analyze_energy_data()
                elif command == "exit":
                    print("Exiting program.")
                    break
                else:
                    print("Invalid command.")
            
            client.loop_stop()
            client.disconnect()
        except Exception as e:
            print(f"Could not connect or receive data from broker: {e}")
            sys.exit(1)

if __name__ == "__main__":
    # Example usage
    app = SmartSustainabilityApp(broker_address="localhost", topic="energy/data")
    app.run()
```

### Key Components:

1. **MQTT Setup**: The program uses the `paho-mqtt` library to connect to an MQTT broker and subscribe to a topic for IoT data.
2. **Data Handling**: Incoming messages are expected to be JSON-encoded with specific fields like `timestamp`, `device_id`, and `energy_consumption`.
3. **Data Analysis**: Uses Pandas for data manipulation, grouping the data by `device_id` for energy analysis.
4. **Visualization**: Employs Matplotlib to plot energy consumption for easier visualization.
5. **Error Handling**: Includes try-except blocks to handle potential errors in JSON decoding, key access, and general exceptions during analysis and plotting.
6. **User Interaction**: Provides a simple interface to analyze the data or exit the loop.

Before using this script, ensure the necessary Python packages (`paho-mqtt`, `pandas`, and `matplotlib`) are installed. The broker address and topic should be configured to match your specific IoT setup. This program does not cover actual implementation details of device integration, which would be highly context-dependent.