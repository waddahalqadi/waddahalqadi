#include <iostream>
#include <cmath>
#include <chrono>
#include <thread>
#include <cppSMTP/smtp.h>

class SmartMeter {
private:
    double previousReading;
    double currentReading;
    double energyConsumed;
    double costPerKWh;
    double maxConsumption;
    bool alertSent;

public:
    SmartMeter(double cost, double maxConsump) : costPerKWh(cost), maxConsumption(maxConsump), previousReading(0.0), currentReading(0.0), energyConsumed(0.0), alertSent(false) {}

    void captureReading(double reading) {
        previousReading = currentReading;
        currentReading = reading;
    }

    void calculateConsumption() {
        double unitsConsumed = currentReading - previousReading;
        energyConsumed += unitsConsumed;

        if (energyConsumed >= maxConsumption && !alertSent) {
            sendAlertEmail();
            alertSent = true;
        }
    }

    void sendAlertEmail() {
        // Set up SMTP configuration
        smtp::SMTPConfig config;
        config.server = "smtp.example.com";
        config.username = "your_username";
        config.password = "your_password";

        // Create an SMTP instance
        smtp::SMTP smtp(config);

        // Compose and send the email
        smtp::SMTPMessage message;
        message.from = "your_email@example.com";
        message.to = "recipient@example.com";
        message.subject = "Alert: Maximum Consumption Reached!";
        message.body = "Your energy consumption has reached the maximum limit.";

        smtp.send(message);
    }

    double getCurrentConsumption() const {
        return energyConsumed;
    }

    double getCurrentCost() const {
        return energyConsumed * costPerKWh;
    }
};

int main() {
    double initialReading = 1000.0; // Initial meter reading
    double costPerKWh = 0.15;       // Cost in JOD per kWh
    double maxConsumption = 100.0;  // Maximum consumption in kWh
    SmartMeter meter(costPerKWh, maxConsumption);

    // Simulate meter readings over time (for demonstration purposes)
    for (int i = 0; i < 10; ++i) {
        double simulatedReading = initialReading + i * 50.0; // Simulated increment
        meter.captureReading(simulatedReading);
        meter.calculateConsumption();

        std::this_thread::sleep_for(std::chrono::seconds(1)); // Simulate time passing

        std::cout << "Current Consumption: " << meter.getCurrentConsumption() << " kWh\n";
        std::cout << "Current Cost: " << meter.getCurrentCost() << " JOD\n";
    }

    return 0;
}
