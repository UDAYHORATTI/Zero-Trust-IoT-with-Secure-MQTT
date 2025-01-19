# Zero-Trust-IoT-with-Secure-MQTT
Make sure to configure your MQTT broker for TLS encryption and use certificates for device authentication.
import paho.mqtt.client as mqtt
import ssl
import json
import time
import random
import string

# MQTT Broker Configuration
MQTT_BROKER = "your-broker.com"
MQTT_PORT = 8883  # Secure MQTT over TLS
MQTT_KEEP_ALIVE_INTERVAL = 60

# Device Information
DEVICE_ID = "Device_A"  # Unique Device ID
CLIENT_CERT_PATH = "path/to/device_cert.pem"
CLIENT_KEY_PATH = "path/to/device_key.pem"
CA_CERT_PATH = "path/to/ca_cert.pem"

# Zero Trust: Authenticate devices using certificates
def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print(f"Device {DEVICE_ID} connected securely to the MQTT broker.")
        client.subscribe(f"iot/{DEVICE_ID}/#")
    else:
        print("Failed to connect, return code", rc)

# Handle incoming messages with ACL check
def on_message(client, userdata, msg):
    print(f"Message received on topic {msg.topic}: {msg.payload.decode()}")
    # Check if the message is from an authorized source
    if not check_access(msg.topic):
        print(f"Unauthorized access attempt detected from device on topic: {msg.topic}")
        return
    print(f"Processing message from topic {msg.topic}: {msg.payload.decode()}")

def check_access(topic):
    """
    Simple ACL check: only allow devices with the proper topic structure.
    In a Zero Trust model, ACLs should be based on device IDs, topic patterns, and more.
    """
    allowed_devices = ["Device_A", "Device_B"]
    if any(device in topic for device in allowed_devices):
        return True
    return False

def secure_publish(client, topic, message):
    """
    Publish messages securely using encrypted MQTT channels.
    """
    if check_access(topic):
        print(f"Publishing to topic: {topic}")
        client.publish(topic, message)
    else:
        print(f"Access Denied to publish on topic: {topic}")

def generate_random_message():
    """
    Generate a random payload message for testing.
    """
    return ''.join(random.choices(string.ascii_uppercase + string.digits, k=20))

if __name__ == "__main__":
    # MQTT Client Setup
    client = mqtt.Client(DEVICE_ID)
    client.tls_set(ca_certs=CA_CERT_PATH, certfile=CLIENT_CERT_PATH, keyfile=CLIENT_KEY_PATH, tls_version=ssl.PROTOCOL_TLSv1_2)
    client.on_connect = on_connect
    client.on_message = on_message

    # Connect to MQTT broker with TLS encryption
    client.connect(MQTT_BROKER, MQTT_PORT, MQTT_KEEP_ALIVE_INTERVAL)

    # Start client loop for message handling
    client.loop_start()

    # Securely publish test messages
    for _ in range(5):
        test_topic = f"iot/{DEVICE_ID}/data"
        test_message = generate_random_message()
        secure_publish(client, test_topic, test_message)
        time.sleep(2)  # Simulate periodic communication

    # Keep the script running to receive messages
    while True:
        time.sleep(5)
