#include <iostream>
#include <vector>
#include <string>
#include <limits>
#include <iomanip>

using namespace std;

// ---------------- FuelRecord Component ----------------
class FuelRecord
{
public:
  string date;
  float litres;
  float odometer;
  float price;

  FuelRecord(string d, float l, float o, float p)
      : date(d), litres(l), odometer(o), price(p) {}
};

// ---------------- MaintenanceRecord Component ----------------
class MaintenanceRecord
{
public:
  string date;
  string type;
  float odometer;
  string notes;

  MaintenanceRecord(string d, string t, float o, string n)
      : date(d), type(t), odometer(o), notes(n) {}
};

// ---------------- Vehicle Component ----------------
class Vehicle
{
private:
  string numberPlate;
  string name;
  string model;
  int year;
  vector<FuelRecord> fuelRecords;
  vector<MaintenanceRecord> maintenanceRecords;

public:
  Vehicle(string plate, string n, string m, int y)
      : numberPlate(plate), name(n), model(m), year(y) {}

  string getNumberPlate() const { return numberPlate; }

  void displayVehicleInfo() const
  {
    cout << "\n+---------------------------------------------+\n";
    cout << "|           VEHICLE INFORMATION               |\n";
    cout << "+---------------------------------------------+\n";
    cout << "| Number Plate : " << setw(30) << left << numberPlate << "|\n";
    cout << "| Name         : " << setw(30) << left << name << "|\n";
    cout << "| Model        : " << setw(30) << left << model << "|\n";
    cout << "| Year         : " << setw(30) << left << year << "|\n";
    cout << "+---------------------------------------------+\n";
  }

  void addFuelRecord(const FuelRecord &record)
  {
    fuelRecords.push_back(record);
  }

  void addMaintenanceRecord(const MaintenanceRecord &record)
  {
    maintenanceRecords.push_back(record);
  }

  void showFuelHistory() const
  {
    cout << "\n=========== FUEL HISTORY: " << numberPlate << " ===========\n";
    if (fuelRecords.empty())
    {
      cout << "No fuel records available.\n";
    }
    else
    {
      for (const FuelRecord &r : fuelRecords)
      {
        cout << "---------------------------------------------\n";
        cout << "Date     : " << r.date << "\n";
        cout << "Litres   : " << r.litres << "\n";
        cout << "Odometer : " << r.odometer << " km\n";
        cout << "Price    : Rs. " << r.price << "\n";
      }
    }
    cout << "---------------------------------------------\n";
  }

  void showMaintenanceHistory() const
  {
    cout << "\n======= MAINTENANCE HISTORY: " << numberPlate << " =======\n";
    if (maintenanceRecords.empty())
    {
      cout << "No maintenance records available.\n";
    }
    else
    {
      for (const MaintenanceRecord &r : maintenanceRecords)
      {
        cout << "---------------------------------------------\n";
        cout << "Date     : " << r.date << "\n";
        cout << "Type     : " << r.type << "\n";
        cout << "Odometer : " << r.odometer << " km\n";
        cout << "Notes    : " << r.notes << "\n";
      }
    }
    cout << "---------------------------------------------\n";
  }

  void calculateFuelAverage() const
  {
    if (fuelRecords.size() < 2)
    {
      cout << "\n[!] Not enough fuel records to calculate average.\n";
      return;
    }

    float totalDistance = fuelRecords.back().odometer - fuelRecords.front().odometer;
    float totalLitres = 0;
    for (int i = 1; i < fuelRecords.size(); ++i)
      totalLitres += fuelRecords[i].litres;

    float average = totalDistance / totalLitres;
    cout << fixed << setprecision(2);
    cout << "\n>>> Average Fuel Economy: " << average << " km/litre\n";
  }

  void showMaintenanceReminder() const
  {
    cout << "\nMaintenance Reminder\n";
    if (maintenanceRecords.empty() || fuelRecords.empty())
    {
      cout << " Not enough data to provide a maintenance reminder.\n";
      return;
    }

    float lastOdo = maintenanceRecords.back().odometer;
    float currentOdo = fuelRecords.back().odometer;
    float diff = currentOdo - lastOdo;

    if (diff >= 5000)
      cout << " Time for your next maintenance!\n";
    else
      cout << " Next maintenance in " << (5000 - diff) << " km.\n";
  }
};

// ---------------- Vehicle Manager ----------------
class VehicleManager
{
private:
  vector<Vehicle> vehicles;

public:
  bool isEmpty() const
  {
    return vehicles.empty();
  }

  Vehicle *findVehicle(const string &plate)
  {
    for (Vehicle &v : vehicles)
      if (v.getNumberPlate() == plate)
        return &v;
    return nullptr;
  }

  void addVehicle(const string &plate, const string &name, const string &model, int year)
  {
    if (findVehicle(plate))
    {
      cout << " Vehicle with this number plate already exists!\n";
      return;
    }
    vehicles.emplace_back(plate, name, model, year);
    cout << "Vehicle added successfully!\n";
  }

  void viewAllVehicles() const
  {
    if (vehicles.empty())
    {
      cout << "\n No vehicles registered.\n";
      return;
    }
    cout << "\n=========== REGISTERED VEHICLES ===========\n";
    for (const Vehicle &v : vehicles)
      v.displayVehicleInfo();
  }

  Vehicle *getVehicleForAction()
  {
    if (isEmpty())
    {
      cout << "No vehicles available.\n";
      return nullptr;
    }

    string plate;
    cout << "Enter Vehicle Number Plate: ";
    getline(cin, plate);
    Vehicle *v = findVehicle(plate);

    if (!v)
    {
      cout << " Vehicle not found.\n";
      return nullptr;
    }
    return v;
  }
};

// ---------------- App Controller ----------------
class App
{
private:
  VehicleManager manager;

  void addVehicleUI()
  {
    string plate, name, model;
    int year;

    cout << "\nEnter Number Plate: ";
    getline(cin, plate);
    cout << "Enter Vehicle Name: ";
    getline(cin, name);
    cout << "Enter Model: ";
    getline(cin, model);
    cout << "Enter Year: ";
    cin >> year;
    cin.ignore();

    manager.addVehicle(plate, name, model, year);
  }

  void addFuelRecordUI()
  {
    Vehicle *v = manager.getVehicleForAction();
    if (!v)
      return;

    string date;
    float litres, odo, price;

    cout << "Enter Date (YYYY-MM-DD): ";
    getline(cin, date);
    cout << "Enter Litres: ";
    cin >> litres;
    cout << "Enter Odometer: ";
    cin >> odo;
    cout << "Enter Price: ";
    cin >> price;
    cin.ignore();

    v->addFuelRecord(FuelRecord(date, litres, odo, price));
    cout << " Fuel record added.\n";
  }

  void addMaintenanceUI()
  {
    Vehicle *v = manager.getVehicleForAction();
    if (!v)
      return;

    string date, type, notes;
    float odo;

    cout << "Enter Date: ";
    getline(cin, date);
    cout << "Enter Type (e.g., Oil Change): ";
    getline(cin, type);
    cout << "Enter Odometer: ";
    cin >> odo;
    cin.ignore();
    cout << "Enter Notes: ";
    getline(cin, notes);

    v->addMaintenanceRecord(MaintenanceRecord(date, type, odo, notes));
    cout << " Maintenance record added.\n";
  }

  void showFuelHistoryUI()
  {
    Vehicle *v = manager.getVehicleForAction();
    if (v)
      v->showFuelHistory();
  }

  void showMaintenanceHistoryUI()
  {
    Vehicle *v = manager.getVehicleForAction();
    if (v)
      v->showMaintenanceHistory();
  }

  void calculateFuelAverageUI()
  {
    Vehicle *v = manager.getVehicleForAction();
    if (v)
      v->calculateFuelAverage();
  }

  void maintenanceReminderUI()
  {
    Vehicle *v = manager.getVehicleForAction();
    if (v)
      v->showMaintenanceReminder();
  }

public:
  void run()
  {
    int choice;
    while (true)
    {
      cout << "\n============================================\n";
      cout << "      VEHICLE TRACKER - MAIN MENU          \n";
      cout << "============================================\n";
      cout << "1. Add Vehicle\n";
      cout << "2. Add Fuel Record\n";
      cout << "3. Add Maintenance Record\n";
      cout << "4. View Fuel History\n";
      cout << "5. View Maintenance History\n";
      cout << "6. Calculate Fuel Average\n";
      cout << "7. Show Maintenance Reminder\n";
      cout << "8. View All Registered Vehicles\n";
      cout << "0. Exit\n";
      cout << "============================================\n";
      cout << "Enter your choice: ";
      cin >> choice;
      cin.ignore(numeric_limits<streamsize>::max(), '\n');

      switch (choice)
      {
      case 1:
        addVehicleUI();
        break;
      case 2:
        addFuelRecordUI();
        break;
      case 3:
        addMaintenanceUI();
        break;
      case 4:
        showFuelHistoryUI();
        break;
      case 5:
        showMaintenanceHistoryUI();
        break;
      case 6:
        calculateFuelAverageUI();
        break;
      case 7:
        maintenanceReminderUI();
        break;
      case 8:
        manager.viewAllVehicles();
        break;
      case 0:
        cout << "\nThank you for using the Vehicle Tracker App. Goodbye!\n";
        return;
      default:
        cout << "Invalid choice. Please try again.(0-8)\n";
      }
    }
  }
};

// ---------------- Main ----------------
int main()
{
  App app;
  app.run();
  return 0;
}
