# Sample-code-for-implementation
#include "mbed.h"
#include "LSM6DSLSensor.h"
#include "HTS221Sensor.h"

// Serial connection setup
UnbufferedSerial pc(USBTX, USBRX, 115200);

// Sensor initialization (modify with actual sensor and pins)
I2C i2c(D14, D15);  // Adjust pins as needed
LSM6DSLSensor acc_gyro(&i2c, LSM6DSL_ACC_GYRO_I2C_ADDRESS_HIGH);
HTS221Sensor hum_temp(&i2c);

// Buffer for incoming data from serial
char buffer[1];

// Flag to check for new data
volatile bool newData = false;

void callback_ex() {
    if (pc.readable()) {
        pc.read(buffer, sizeof(buffer));
        newData = true;
    }
}

void print_sensor_data(char command) {
    switch (command) {
        case 'a': // Accelerometer data
            int16_t data[3];
            acc_gyro.get_x_axes(data);
            printf("Accelerometer: X=%d, Y=%d, Z=%d\r\n", data[0], data[1], data[2]);
            break;
        case 't': // Temperature data
            float temperature;
            hum_temp.get_temperature(&temperature);
            printf("Temperature: %.2f C\r\n", temperature);
            break;
        // Add cases for other sensors as needed
        default:
            printf("Unknown command\r\n");
    }
}

int main() {
    pc.set_format(8, BufferedSerial::None, 1);
    pc.attach(&callback_ex, SerialBase::RxIrq);

    // Sensor setup
    acc_gyro.init();
    hum_temp.init();

    while (true) {
        if (newData) {
            print_sensor_data(buffer[0]);
            newData = false;
        }
        ThisThread::sleep_for(100ms); // Reduce CPU usage
    }
}
