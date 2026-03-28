#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
class Phone { //класс phone
private:
    static int count;  //счетчик объектов
    string surname;
    string name;
    string patronymic;
    string address;
    string number;
    int local_time;
    int long_distance_time;
public:
    //три конструктора
    Phone() : surname(""), name(""), patronymic(""), address(""),
        number(""), local_time(0), long_distance_time(0) { //конструктор по умолчанию; инициализирует все поля пустыми строка и нулями
        count++;
        cout << "Created. Total: " << count << endl;
    }
    Phone(string s, string n, string p, string a, string num, int lt, int ldt)
        : surname(s), name(n), patronymic(p), address(a), number(num),
        local_time(lt), long_distance_time(ldt) { //конструктор с параметрами; инициализирует поля переданными значениями
        count++;
        cout << "Created. Total: " << count << endl;
    }
    Phone(const Phone& other)
        : surname(other.surname), name(other.name), patronymic(other.patronymic),
        address(other.address), number(other.number),
        local_time(other.local_time), long_distance_time(other.long_distance_time) { //конструктор копирования; создает объект как копию другого объекта
        count++;
        cout << "Created (copy). Total: " << count << endl;
    }
    //деструктор
    ~Phone() { //вызывается при уничтожении объектов 
        count--;
        cout << "Deleted. Remaining: " << count << endl;
    }

    //методы доступа, устанавливают фамилию имя и тд
    void set_surname(string s) { surname = s; }
    void set_name(string n) { name = n; }
    void set_patronymic(string p) { patronymic = p; }
    void set_address(string a) { address = a; }
    void set_number(string num) { number = num; }
    void set_local_time(int lt) { local_time = lt; }
    void set_long_distance_time(int ldt) { long_distance_time = ldt; }
    string get_surname() { return surname; }
    int get_local_time() { return local_time; }
    int get_long_distance_time() { return long_distance_time; }

    //метод подсчета суммарного времени
    int total_time() {
        return local_time + long_distance_time;
    }

    //метод вывода информации об абоненте
    void show() {
        cout << surname << " " << name << " " << patronymic << " | "
            << address << " | " << number << " | "
            << "Local: " << local_time << " | "
            << "Long: " << long_distance_time << " | "
            << "Total: " << total_time() << endl;
    }
    static int getCount() { return count; } //метод для получения кол-ва объектов
};
int Phone::count = 0;

//внешние функции
//модификация объекта
void modify_object(Phone& obj, int lt, int ldt) { //функция для модификации объекта по ссылке
    cout << "\n--- modify_object (changes original) ---" << endl;
    //новые локальное и междугороднее времена:
    obj.set_local_time(lt);
    obj.set_long_distance_time(ldt);
    cout << "New times: local=" << lt << ", long=" << ldt << endl;
}
//функция попытки модификации копии объекта
void try_to_modify_object(Phone obj, int lt, int ldt) {
    cout << "\n--- try_to_modify_object (changes copy) ---" << endl;

    //новое время у копии
    obj.set_local_time(lt);
    obj.set_long_distance_time(ldt);
    cout << "Inside function: ";
    obj.show();
}

//главная функция
int main() {
    setlocale(LC_ALL, "RU");
    cout << "========== PART 1: CONSTRUCTORS ==========\n" << endl;
    // Демонстрация 3 конструкторов
    Phone p1; //default
    Phone p2("Ivanov", "Ivan", "Ivanovich", "Moscow", "123-45-67", 120, 30);  // parameterized
    Phone p3(p2); // copy
    cout << "\nCurrent objects: " << Phone::getCount() << endl;
    cout << "\n========== PART 2: MODIFY FUNCTIONS ==========\n" << endl;

    // Демонстрация функций
    Phone test("Test", "Testovich", "Test", "City", "000-00-00", 100, 50);
    cout << "Original: ";
    test.show();

    try_to_modify_object(test, 200, 75);
    cout << "After try_to_modify: ";
    test.show(); //не изменился

    modify_object(test, 300, 100);
    cout << "After modify: ";
    test.show(); //изменился

    cout << "\n========== PART 3: CLASSIC OBJECTS ==========\n" << endl;

    //создание файла
    ofstream fout("phones.txt");
    fout << "Ivanov,Ivan,Ivanovich,Moscow,123-45-67,120,30\n"
        << "Petrov,Petr,Petrovich,SPb,234-56-78,50,10\n"
        << "Sidorov,Sidor,Sidorovich,Kazan,345-67-89,200,0\n"
        << "Smirnov,Alexey,Ivanovich,Moscow,456-78-90,80,20\n"
        << "Kuznetsov,Nikolai,Petrovich,Tver,567-89-01,30,5\n";
    fout.close();

    // Загрузка данных в вектор (классические объекты)
    vector<Phone> phones;
    ifstream fin("phones.txt");
    string line;

    while (getline(fin, line)) {
        int p1 = line.find(',');
        int p2 = line.find(',', p1 + 1);
        int p3 = line.find(',', p2 + 1);
        int p4 = line.find(',', p3 + 1);
        int p5 = line.find(',', p4 + 1);
        int p6 = line.find(',', p5 + 1);

        string s = line.substr(0, p1);
        string n = line.substr(p1 + 1, p2 - p1 - 1);
        string patr = line.substr(p2 + 1, p3 - p2 - 1);
        string addr = line.substr(p3 + 1, p4 - p3 - 1);
        string num = line.substr(p4 + 1, p5 - p4 - 1);
        int lt = stoi(line.substr(p5 + 1, p6 - p5 - 1));
        int ldt = stoi(line.substr(p6 + 1));

        phones.push_back(Phone(s, n, patr, addr, num, lt, ldt));
    }
    fin.close();

    // Запросы
    int threshold;
    cout << "Enter local time threshold: ";
    cin >> threshold;

    // а) абоненты с внутригородским временем > заданного
    cout << "\na) Local time > " << threshold << ":" << endl;
    for (int i = 0; i < phones.size(); i++) {
        if (phones[i].get_local_time() > threshold) {
            phones[i].show();
        }
    }

    // б) абоненты, воспользовавшиеся междугородней связью
    cout << "\nb) Used long-distance connection:" << endl;
    for (int i = 0; i < phones.size(); i++) {
        if (phones[i].get_long_distance_time() > 0) {
            phones[i].show();
        }
    }

    // в) алфавитный порядок
    cout << "\nc) Alphabetical order:" << endl;
    vector<Phone> sorted = phones;
    for (int i = 0; i < sorted.size() - 1; i++) {
        for (int j = 0; j < sorted.size() - i - 1; j++) {
            if (sorted[j].get_surname() > sorted[j + 1].get_surname()) {
                Phone temp = sorted[j];
                sorted[j] = sorted[j + 1];
                sorted[j + 1] = temp;
            }
        }
    }
    for (int i = 0; i < sorted.size(); i++) {
        sorted[i].show();
    }

    cout << "\n========== PART 4: DYNAMIC OBJECTS ==========\n" << endl;

    // Динамическое создание объектов (через new)
    vector<Phone*> dynamic_phones;

    ifstream fin2("phones.txt");
    while (getline(fin2, line)) {
        int p1 = line.find(',');
        int p2 = line.find(',', p1 + 1);
        int p3 = line.find(',', p2 + 1);
        int p4 = line.find(',', p3 + 1);
        int p5 = line.find(',', p4 + 1);
        int p6 = line.find(',', p5 + 1);

        string s = line.substr(0, p1);
        string n = line.substr(p1 + 1, p2 - p1 - 1);
        string patr = line.substr(p2 + 1, p3 - p2 - 1);
        string addr = line.substr(p3 + 1, p4 - p3 - 1);
        string num = line.substr(p4 + 1, p5 - p4 - 1);
        int lt = stoi(line.substr(p5 + 1, p6 - p5 - 1));
        int ldt = stoi(line.substr(p6 + 1));

        dynamic_phones.push_back(new Phone(s, n, patr, addr, num, lt, ldt));
    }
    fin2.close();

    cout << "Dynamic objects created. Total: " << Phone::getCount() << endl;

    // Те же запросы для динамических объектов
    cout << "\na) Local time > " << threshold << ":" << endl;
    for (int i = 0; i < dynamic_phones.size(); i++) {
        if (dynamic_phones[i]->get_local_time() > threshold) {
            dynamic_phones[i]->show();
        }
    }

    cout << "\nb) Used long-distance connection:" << endl;
    for (int i = 0; i < dynamic_phones.size(); i++) {
        if (dynamic_phones[i]->get_long_distance_time() > 0) {
            dynamic_phones[i]->show();
        }
    }

    cout << "\nc) Alphabetical order:" << endl;
    for (int i = 0; i < dynamic_phones.size() - 1; i++) {
        for (int j = 0; j < dynamic_phones.size() - i - 1; j++) {
            if (dynamic_phones[j]->get_surname() > dynamic_phones[j + 1]->get_surname()) {
                Phone* temp = dynamic_phones[j];
                dynamic_phones[j] = dynamic_phones[j + 1];
                dynamic_phones[j + 1] = temp;
            }
        }
    }
    for (int i = 0; i < dynamic_phones.size(); i++) {
        dynamic_phones[i]->show();
    }

    // Удаление динамических объектов
    cout << "\n--- Deleting dynamic objects ---" << endl;
    for (int i = 0; i < dynamic_phones.size(); i++) {
        delete dynamic_phones[i];
    }

    cout << "\nFinal objects count: " << Phone::getCount() << endl;

    return 0;
}
