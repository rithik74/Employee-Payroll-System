# Employee-Payroll-System
#include <bits/stdc++.h>
using namespace std;

// -------------------- BASE CLASS --------------------
class Employee {
protected:
    string name, id;
    double basicSalary;

public:
    Employee(string name="", string id="", double basicSalary=0)
        : name(name), id(id), basicSalary(basicSalary) {}

    virtual double calculateSalary() const = 0;

    virtual void display() const {
        cout << "ID: " << id
             << "\nName: " << name
             << "\nBasic Salary: " << basicSalary << endl;
    }

    virtual string serialize() const {
        return id + " " + name + " " + to_string(basicSalary);
    }

    string getId() const { return id; }
};


// -------------------- PERMANENT EMPLOYEE --------------------
class PermanentEmployee : public Employee {
    double hra, da;  // House Rent Allowance, Dearness Allowance
public:
    PermanentEmployee(string name, string id, double basicSalary, double hra, double da)
        : Employee(name, id, basicSalary), hra(hra), da(da) {}

    double calculateSalary() const override {
        return basicSalary + hra + da;
    }

    void display() const override {
        cout << "\n[Permanent Employee]\n";
        Employee::display();
        cout << "HRA: " << hra
             << "\nDA: " << da
             << "\nNet Salary: " << calculateSalary() << endl;
    }

    string serialize() const override {
        return "P " + Employee::serialize() + " " + to_string(hra) + " " + to_string(da);
    }
};


// -------------------- CONTRACT EMPLOYEE --------------------
class ContractEmployee : public Employee {
    double hourlyRate;
    int hoursWorked;

public:
    ContractEmployee(string name, string id, double hourlyRate, int hoursWorked)
        : Employee(name, id, 0), hourlyRate(hourlyRate), hoursWorked(hoursWorked) {}

    double calculateSalary() const override {
        return hourlyRate * hoursWorked;
    }

    void display() const override {
        cout << "\n[Contract Employee]\n";
        cout << "ID: " << id
             << "\nName: " << name
             << "\nHourly Rate: " << hourlyRate
             << "\nHours Worked: " << hoursWorked
             << "\nNet Salary: " << calculateSalary() << endl;
    }

    string serialize() const override {
        return "C " + id + " " + name + " " + to_string(hourlyRate) + " " + to_string(hoursWorked);
    }
};


// -------------------- PAYROLL MANAGER --------------------
class PayrollSystem {
    vector<Employee*> employees;

public:

    void addEmployee(Employee* emp) {
        employees.push_back(emp);
        cout << "\nEmployee Added Successfully!\n";
    }

    void displayAll() const {
        cout << "\n=========== EMPLOYEE LIST ===========\n";
        for (auto e : employees)
            e->display();
    }

    Employee* searchById(const string& empId) {
        for (auto e : employees)
            if (e->getId() == empId)
                return e;

        return nullptr;
    }

    void saveToFile(string filename) {
        ofstream out(filename);
        if (!out) throw runtime_error("Cannot open file!");

        for (auto e : employees)
            out << e->serialize() << endl;

        out.close();
        cout << "\nData Saved Successfully to " << filename << endl;
    }

    void loadFromFile(string filename) {
        ifstream in(filename);
        if (!in) throw runtime_error("File not found!");

        employees.clear();
        string type, id, name;
        double a, b;
        int c;

        while (in >> type) {
            if (type == "P") {
                in >> id >> name >> a >> b >> c;
                employees.push_back(new PermanentEmployee(name, id, a, b, c));
            }
            else if (type == "C") {
                in >> id >> name >> a >> c;
                employees.push_back(new ContractEmployee(name, id, a, c));
            }
        }
        in.close();
        cout << "\nData Loaded Successfully from " << filename << endl;
    }

    ~PayrollSystem() {
        for (auto e : employees)
            delete e;
    }
};


// -------------------- MAIN MENU --------------------
int main() {
    PayrollSystem ps;

    int choice;
    while (true) {
        cout << "\n=========== PAYROLL MANAGEMENT SYSTEM ===========\n";
        cout << "1. Add Permanent Employee\n";
        cout << "2. Add Contract Employee\n";
        cout << "3. Display All Employees\n";
        cout << "4. Search Employee by ID\n";
        cout << "5. Save to File\n";
        cout << "6. Load from File\n";
        cout << "0. Exit\n";
        cout << "Enter choice: ";
        cin >> choice;

        if (choice == 0) break;

        string name, id;
        double s1, s2;
        int h;

        switch (choice) {
        case 1:
            cout << "Enter ID, Name, Basic Salary, HRA, DA: ";
            cin >> id >> name >> s1 >> s2 >> h;
            ps.addEmployee(new PermanentEmployee(name, id, s1, s2, h));
            break;

        case 2:
            cout << "Enter ID, Name, Hourly Rate, Hours Worked: ";
            cin >> id >> name >> s1 >> h;
            ps.addEmployee(new ContractEmployee(name, id, s1, h));
            break;

        case 3:
            ps.displayAll();
            break;

        case 4: {
            cout << "Enter Employee ID: ";
            cin >> id;
            Employee* emp = ps.searchById(id);
            if (emp) emp->display();
            else cout << "Employee not found!\n";
            break;
        }

        case 5:
            ps.saveToFile("payroll.txt");
            break;

        case 6:
            ps.loadFromFile("payroll.txt");
            break;

        default:
            cout << "Invalid choice!\n";
        }
    }

    return 0;
}
