#include <iostream>
#include <algorithm>
#include <chrono>
#include <cstdlib>
#include <ctime>
#include <iomanip>
#include <string>
#include <vector>
using namespace std;
using namespace std::chrono;

// Exercise 1
namespace ex1 {
int arraySum(int A[], int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) {
        sum += A[i];
    }
    return sum;
}
void run() {
    srand(time_t(0));
    const int RUNS = 5;
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    cout << left << setw(12) << "n" << "Avg time (us)" << endl;
    for (int n : sizes) {
        int* A = new int[n];
        for (int i = 0; i < n; i++) A[i] = rand() % 100;
        double total = 0;
        long long keep = 0;
        for (int r = 0; r < RUNS; r++) {
            auto start = high_resolution_clock::now();
            keep += arraySum(A, n);
            auto stop = high_resolution_clock::now();
            total += duration<double, micro>(stop - start).count();
        }
        cout << left << setw(12) << n << fixed << setprecision(2)
             << total / RUNS << "   (checksum " << keep << ")" << endl;
        delete[] A;
    }
}
}
// Exercise 2
namespace ex2 {
int linearSearch(int A[], int n, int key) {
    for (int i = 0; i < n; i++) {
        if (A[i] == key) return i;
    }
    return -1;
}
double timeSearch(int A[], int n, int key) {
    const int RUNS = 5;
    double total = 0;
    volatile int sink;
    for (int r = 0; r < RUNS; r++) {
        auto start = high_resolution_clock::now();
        sink = linearSearch(A, n, key);
        auto stop = high_resolution_clock::now();
        total += duration<double, micro>(stop - start).count();
    }
    (void)sink;
    return total / RUNS;
}
void run() {
    srand(time_t(0));
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    cout << left << setw(10) << "n" << setw(14) << "Best (us)"
         << setw(14) << "Avg (us)" << "Worst (us)" << endl;
    for (int n : sizes) {
        int* A = new int[n];
        for (int i = 0; i < n; i++) A[i] = i;
        for (int i = n - 1; i > 0; i--) { int j = rand() % (i + 1); swap(A[i], A[j]); }
        double best  = timeSearch(A, n, A[0]);
        double avg   = timeSearch(A, n, A[n / 2]);
        double worst = timeSearch(A, n, -999999);
        cout << fixed << setprecision(2) << left << setw(10) << n
             << setw(14) << best << setw(14) << avg << worst << endl;
        delete[] A;
    }
}
}
// Exercise 3
namespace ex3 {
int arraySum(int A[], int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) sum += A[i];
    return sum;
}
void bubbleSort(int A[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - 1 - i; j++) {
            if (A[j] > A[j + 1]) {
                int temp = A[j];
                A[j] = A[j + 1];
                A[j + 1] = temp;
            }
        }
    }
}
void run() {
    srand(42);
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    cout << left << setw(10) << "n" << setw(20) << "Sum time (us)"
         << "Bubble sort time (us)" << endl;
    for (int n : sizes) {
        int* data = new int[n];
        for (int i = 0; i < n; i++) data[i] = rand();
        double sumTotal = 0;
        volatile int sink;
        for (int r = 0; r < 5; r++) {
            auto s = high_resolution_clock::now();
            sink = arraySum(data, n);
            auto e = high_resolution_clock::now();
            sumTotal += duration<double, micro>(e - s).count();
        }
        (void)sink;
        int* copy = new int[n];
        for (int i = 0; i < n; i++) copy[i] = data[i];
        auto start = high_resolution_clock::now();
        bubbleSort(copy, n);
        auto stop = high_resolution_clock::now();
        double bubble = duration<double, micro>(stop - start).count();
        cout << fixed << setprecision(2) << left << setw(10) << n
             << setw(20) << sumTotal / 5 << bubble << endl;
        delete[] data;
        delete[] copy;
    }
}
}
// Exercise 4
namespace ex4 {
const int CAP = 100000;
int A[CAP];
void run() {
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    const int RUNS = 5;
    cout << "sizeof(A) = " << sizeof(A) << " bytes (fixed, whatever n is)\n" << endl;
    cout << left << setw(10) << "n" << setw(20) << "Static fill (us)"
    << setw(20) << "Dynamic fill (us)" << "Bytes (dynamic)" << endl;
    for (int n : sizes) {
        int* B = new int[n];
        double tStatic = 0, tDynamic = 0;
        for (int r = 0; r < RUNS; r++) {
            auto s1 = high_resolution_clock::now();
            for (int i = 0; i < n; i++) A[i] = i;
            auto e1 = high_resolution_clock::now();
            tStatic += duration<double, micro>(e1 - s1).count();
            auto s2 = high_resolution_clock::now();
            for (int i = 0; i < n; i++) B[i] = i;
            auto e2 = high_resolution_clock::now();
            tDynamic += duration<double, micro>(e2 - s2).count();
        }
        cout << fixed << setprecision(2) << left << setw(10) << n
        << setw(20) << tStatic / RUNS << setw(20) << tDynamic / RUNS
        << n * sizeof(int) << endl;
        delete[] B;
    }
}
}
// Exercise 5
namespace ex5 {
void run() {
    srand(42);
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    const int RUNS = 5;
    {
        vector<int> v;
        size_t lastCap = v.capacity();
        cout << "Capacity changes while pushing 100000 elements:" << endl;
        for (int i = 0; i < 100000; i++) {
            v.push_back(rand());
            if (v.capacity() != lastCap) {
                cout << "  size " << setw(6) << v.size()
                << " -> capacity " << v.capacity() << endl;
                lastCap = v.capacity();
            }
        }
        cout << endl;
    }
    cout << left << setw(10) << "n" << setw(24) << "push_back only (us)"
    << setw(28) << "reserve + push_back (us)" << "Capacity jumps" << endl;
    for (int n : sizes) {
        double tPlain = 0, tReserve = 0;
        int jumps = 0;
        for (int r = 0; r < RUNS; r++) {
            auto s1 = high_resolution_clock::now();
            vector<int> v;
            size_t cap = v.capacity();
            int j = 0;
            for (int i = 0; i < n; i++) {
                v.push_back(rand());
                if (v.capacity() != cap) { cap = v.capacity(); j++; }
            }
            auto e1 = high_resolution_clock::now();
            tPlain += duration<double, micro>(e1 - s1).count();
            jumps = j;
            auto s2 = high_resolution_clock::now();
            vector<int> w;
            w.reserve(n);
            for (int i = 0; i < n; i++) w.push_back(rand());
            auto e2 = high_resolution_clock::now();
            tReserve += duration<double, micro>(e2 - s2).count();
        }
        cout << fixed << setprecision(2) << left << setw(10) << n
        << setw(24) << tPlain / RUNS << setw(28) << tReserve / RUNS
        << jumps << endl;
    }
}
}
// Challenge 1
namespace c1 {
void bubbleSort(int A[], int n) {
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - 1 - i; j++)
            if (A[j] > A[j + 1]) { int t = A[j]; A[j] = A[j + 1]; A[j + 1] = t; }
}
void run() {
    srand(42);
    int sizes[] = {1000, 2000, 5000, 10000, 50000, 100000};
    cout << left << setw(10) << "n" << setw(20) << "Bubble sort (ms)"
    << setw(20) << "std::sort (ms)" << "Ratio" << endl;
    for (int n : sizes) {
        int* base = new int[n];
        for (int i = 0; i < n; i++) base[i] = rand();
        int* copy1 = new int[n];
        int* copy2 = new int[n];
        copy(base, base + n, copy1);
        copy(base, base + n, copy2);
        auto s1 = high_resolution_clock::now();
        bubbleSort(copy1, n);
        auto e1 = high_resolution_clock::now();
        double tBubble = duration<double, milli>(e1 - s1).count();
        auto s2 = high_resolution_clock::now();
        sort(copy2, copy2 + n);
        auto e2 = high_resolution_clock::now();
        double tStd = duration<double, milli>(e2 - s2).count();
        cout << fixed << setprecision(3) << left << setw(10) << n
        << setw(20) << tBubble << setw(20) << tStd
        << setprecision(1) << tBubble / tStd << endl;
        delete[] base; delete[] copy1; delete[] copy2;
    }
}
}
// Challenge 2
namespace c2 {
typedef vector<vector<int> > Matrix;
Matrix makeRandom(int n) {
    Matrix M(n, vector<int>(n));
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            M[i][j] = rand() % 10;
    return M;
}
void multiply(const Matrix& X, const Matrix& Y, Matrix& Z, int n) {
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) {
            int sum = 0;
            for (int k = 0; k < n; k++)
                sum += X[i][k] * Y[k][j];
            Z[i][j] = sum;
        }
}
void run() {
    srand(42);
    int sizes[] = {50, 100, 200, 400, 800};
    cout << left << setw(8) << "n" << setw(20) << "Multiply (ms)"
    << setw(20) << "Approx. bytes" << "Time ratio vs previous" << endl;
    double prev = 0;
    for (int n : sizes) {
        Matrix X = makeRandom(n), Y = makeRandom(n);
        Matrix Z(n, vector<int>(n, 0));
        auto start = high_resolution_clock::now();
        multiply(X, Y, Z, n);
        auto stop = high_resolution_clock::now();
        double ms = duration<double, milli>(stop - start).count();
        cout << fixed << setprecision(3) << left << setw(8) << n
        << setw(20) << ms << setw(20) << 3LL * n * n * sizeof(int);
        if (prev > 0) cout << setprecision(2) << ms / prev << "x";
        else cout << "-";
        cout << endl;
        prev = ms;
    }
}
}
// Challenge 3
namespace c3 {
long long fibRecursive(int n) {
    if (n <= 1) return n;
    return fibRecursive(n - 1) + fibRecursive(n - 2);
}
unsigned long long fibIterative(int n) {
    unsigned long long a = 0, b = 1;
    for (int i = 0; i < n; i++) {
        unsigned long long next = a + b;
        a = b;
        b = next;
    }
    return a;
}
void run() {
    cout << "--- Recursive (naive) ---" << endl;
    cout << left << setw(10) << "n" << setw(16) << "Time (ms)" << "Ratio vs previous" << endl;
    int rn[] = {20, 25, 30, 35};
    double prev = 0;
    for (int n : rn) {
        auto s = high_resolution_clock::now();
        volatile long long r = fibRecursive(n);
        auto e = high_resolution_clock::now();
        (void)r;
        double ms = duration<double, milli>(e - s).count();
        cout << fixed << setprecision(3) << left << setw(10) << n << setw(16) << ms;
        if (prev > 0) cout << setprecision(1) << ms / prev << "x";
        cout << endl;
        prev = ms;
    }
    cout << "\n--- Iterative ---" << endl;
    cout << left << setw(12) << "n" << "Time (us)" << endl;
    int in[] = {20, 35, 1000, 1000000};
    for (int n : in) {
        auto s = high_resolution_clock::now();
        volatile unsigned long long r = fibIterative(n);
        auto e = high_resolution_clock::now();
        (void)r;
        cout << fixed << setprecision(3) << left << setw(12) << n
        << duration<double, micro>(e - s).count() << endl;
    }
}
}
// Challenge 4
namespace c4 {
struct Node {
    int data;
    Node* next;
};
int searchArray(const int* A, int n, int key) {
    for (int i = 0; i < n; i++) if (A[i] == key) return i;
    return -1;
}
int searchVector(const vector<int>& v, int key) {
    for (size_t i = 0; i < v.size(); i++) if (v[i] == key) return (int)i;
    return -1;
}
int searchList(Node* head, int key) {
    int idx = 0;
    for (Node* p = head; p != nullptr; p = p->next, idx++) if (p->data == key) return idx;
    return -1;
}
void run() {
    const int n = 100000;
    const int key = n - 10;
    volatile int sink;
    auto s = high_resolution_clock::now();
    vector<int> v;
    for (int i = 0; i < n; i++) v.push_back(i);
    auto e = high_resolution_clock::now();
    double vIns = duration<double, micro>(e - s).count();
    s = high_resolution_clock::now();
    sink = searchVector(v, key);
    e = high_resolution_clock::now();
    double vSearch = duration<double, micro>(e - s).count();
    size_t vBytes = v.capacity() * sizeof(int);
    s = high_resolution_clock::now();
    int* arr = new int[n];
    for (int i = 0; i < n; i++) arr[i] = i;
    e = high_resolution_clock::now();
    double aIns = duration<double, micro>(e - s).count();
    s = high_resolution_clock::now();
    sink = searchArray(arr, n, key);
    e = high_resolution_clock::now();
    double aSearch = duration<double, micro>(e - s).count();
    size_t aBytes = n * sizeof(int);
    s = high_resolution_clock::now();
    Node* head = nullptr;
    Node* tail = nullptr;
    for (int i = 0; i < n; i++) {
        Node* node = new Node{i, nullptr};
        if (head == nullptr) head = tail = node;
        else { tail->next = node; tail = node; }
    }
    e = high_resolution_clock::now();
    double lIns = duration<double, micro>(e - s).count();
    s = high_resolution_clock::now();
    sink = searchList(head, key);
    e = high_resolution_clock::now();
    double lSearch = duration<double, micro>(e - s).count();
    size_t lBytes = n * sizeof(Node);
    (void)sink;
    cout << "Node size: " << sizeof(Node) << " bytes" << endl << endl;
    cout << left << setw(14) << "Structure" << setw(16) << "Insert (us)"
    << setw(16) << "Search (us)" << setw(16) << "Bytes/element" << "Total bytes" << endl;
    cout << fixed << setprecision(1);
    cout << left << setw(14) << "vector"      << setw(16) << vIns << setw(16) << vSearch
    << setw(16) << (double)vBytes / n << vBytes << endl;
    cout << left << setw(14) << "raw array"   << setw(16) << aIns << setw(16) << aSearch
    << setw(16) << (double)aBytes / n << aBytes << endl;
    cout << left << setw(14) << "linked list" << setw(16) << lIns << setw(16) << lSearch
    << setw(16) << (double)lBytes / n << lBytes << endl;
    delete[] arr;
    while (head != nullptr) {
        Node* next = head->next;
        delete head;
        head = next;
    }
}
}
// Menu
int main(int argc, char* argv[]) {
    string choice;
    if (argc > 1) {
        choice = argv[1];
    } else {
        cout << "CS111 Week 3 Lab - choose an experiment:\n"
        "  1   Exercise 1\n"
        "  2   Exercise 2\n"
        "  3   Exercise 3\n"
        "  4   Exercise 4\n"
        "  5   Exercise 5\n"
        "  c1  Challenge 1\n"
        "  c2  Challenge 2\n"
        "  c3  Challenge 3\n"
        "  c4  Challenge 4\n"
        "  all  Run everything\n"
        "> ";
        cin >> choice;
    }
    bool ran = false;
    auto hdr = [](const char* t) { cout << "\n===== " << t << " =====\n"; };
    bool all = (choice == "all");
    if (all || choice == "1") { hdr("Exercise 1"); ex1::run(); ran = true; }
    if (all || choice == "2") { hdr("Exercise 2"); ex2::run(); ran = true; }
    if (all || choice == "3") { hdr("Exercise 3"); ex3::run(); ran = true; }
    if (all || choice == "4") { hdr("Exercise 4"); ex4::run(); ran = true; }
    if (all || choice == "5") { hdr("Exercise 5 "); ex5::run(); ran = true; }
    if (all || choice == "c1") { hdr("Challenge 1"); c1::run(); ran = true; }
    if (all || choice == "c2") { hdr("Challenge 2"); c2::run(); ran = true; }
    if (all || choice == "c3") { hdr("Challenge 3"); c3::run(); ran = true; }
    if (all || choice == "c4") { hdr("Challenge 4"); c4::run(); ran = true; }
    if (!ran) cout << "Unknown choice: " << choice << endl;
    return 0;
}
#include <iostream>
#include <algorithm>
#include <chrono>
#include <cstdlib>
#include <ctime>
#include <iomanip>
#include <string>
#include <vector>
using namespace std;
using namespace std::chrono;
// Exercise 1
namespace ex1 {
int arraySum(int A[], int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) {
        sum += A[i];
    }
    return sum;
}
void run() {
    srand(time_t(0));
    const int RUNS = 5;
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    cout << left << setw(12) << "n" << "Avg time (us)" << endl;
    for (int n : sizes) {
        int* A = new int[n];
        for (int i = 0; i < n; i++) A[i] = rand() % 100;
        double total = 0;
        long long keep = 0;
        for (int r = 0; r < RUNS; r++) {
            auto start = high_resolution_clock::now();
            keep += arraySum(A, n);
            auto stop = high_resolution_clock::now();
            total += duration<double, micro>(stop - start).count();
        }
        cout << left << setw(12) << n << fixed << setprecision(2)
             << total / RUNS << "   (checksum " << keep << ")" << endl;
        delete[] A;
    }
}
}
// Exercise 2
namespace ex2 {
int linearSearch(int A[], int n, int key) {
    for (int i = 0; i < n; i++) {
        if (A[i] == key) return i;
    }
    return -1;
}
double timeSearch(int A[], int n, int key) {
    const int RUNS = 5;
    double total = 0;
    volatile int sink;
    for (int r = 0; r < RUNS; r++) {
        auto start = high_resolution_clock::now();
        sink = linearSearch(A, n, key);
        auto stop = high_resolution_clock::now();
        total += duration<double, micro>(stop - start).count();
    }
    (void)sink;
    return total / RUNS;
}
void run() {
    srand(time_t(0));
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    cout << left << setw(10) << "n" << setw(14) << "Best (us)"
         << setw(14) << "Avg (us)" << "Worst (us)" << endl;
    for (int n : sizes) {
        int* A = new int[n];
        for (int i = 0; i < n; i++) A[i] = i;
        for (int i = n - 1; i > 0; i--) { int j = rand() % (i + 1); swap(A[i], A[j]); }
        double best  = timeSearch(A, n, A[0]);
        double avg   = timeSearch(A, n, A[n / 2]);
        double worst = timeSearch(A, n, -999999);
        cout << fixed << setprecision(2) << left << setw(10) << n
             << setw(14) << best << setw(14) << avg << worst << endl;
        delete[] A;
    }
}
}
// Exercise 3
namespace ex3 {
int arraySum(int A[], int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) sum += A[i];
    return sum;
}
void bubbleSort(int A[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - 1 - i; j++) {
            if (A[j] > A[j + 1]) {
                int temp = A[j];
                A[j] = A[j + 1];
                A[j + 1] = temp;
            }
        }
    }
}
void run() {
    srand(42);
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    cout << left << setw(10) << "n" << setw(20) << "Sum time (us)"
         << "Bubble sort time (us)" << endl;
    for (int n : sizes) {
        int* data = new int[n];
        for (int i = 0; i < n; i++) data[i] = rand();
        double sumTotal = 0;
        volatile int sink;
        for (int r = 0; r < 5; r++) {
            auto s = high_resolution_clock::now();
            sink = arraySum(data, n);
            auto e = high_resolution_clock::now();
            sumTotal += duration<double, micro>(e - s).count();
        }
        (void)sink;
        int* copy = new int[n];
        for (int i = 0; i < n; i++) copy[i] = data[i];
        auto start = high_resolution_clock::now();
        bubbleSort(copy, n);
        auto stop = high_resolution_clock::now();
        double bubble = duration<double, micro>(stop - start).count();
        cout << fixed << setprecision(2) << left << setw(10) << n
             << setw(20) << sumTotal / 5 << bubble << endl;
        delete[] data;
        delete[] copy;
    }
}
}
// Exercise 4
namespace ex4 {
const int CAP = 100000;
int A[CAP];
void run() {
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    const int RUNS = 5;
    cout << "sizeof(A) = " << sizeof(A) << " bytes (fixed, whatever n is)\n" << endl;
    cout << left << setw(10) << "n" << setw(20) << "Static fill (us)"
    << setw(20) << "Dynamic fill (us)" << "Bytes (dynamic)" << endl;
    for (int n : sizes) {
        int* B = new int[n];
        double tStatic = 0, tDynamic = 0;
        for (int r = 0; r < RUNS; r++) {
            auto s1 = high_resolution_clock::now();
            for (int i = 0; i < n; i++) A[i] = i;
            auto e1 = high_resolution_clock::now();
            tStatic += duration<double, micro>(e1 - s1).count();
            auto s2 = high_resolution_clock::now();
            for (int i = 0; i < n; i++) B[i] = i;
            auto e2 = high_resolution_clock::now();
            tDynamic += duration<double, micro>(e2 - s2).count();
        }
        cout << fixed << setprecision(2) << left << setw(10) << n
        << setw(20) << tStatic / RUNS << setw(20) << tDynamic / RUNS
        << n * sizeof(int) << endl;
        delete[] B;
    }
}
}
// Exercise 5
namespace ex5 {
void run() {
    srand(42);
    int sizes[] = {1000, 5000, 10000, 50000, 100000};
    const int RUNS = 5;
    {
        vector<int> v;
        size_t lastCap = v.capacity();
        cout << "Capacity changes while pushing 100000 elements:" << endl;
        for (int i = 0; i < 100000; i++) {
            v.push_back(rand());
            if (v.capacity() != lastCap) {
                cout << "  size " << setw(6) << v.size()
                << " -> capacity " << v.capacity() << endl;
                lastCap = v.capacity();
            }
        }
        cout << endl;
    }
    cout << left << setw(10) << "n" << setw(24) << "push_back only (us)"
    << setw(28) << "reserve + push_back (us)" << "Capacity jumps" << endl;
    for (int n : sizes) {
        double tPlain = 0, tReserve = 0;
        int jumps = 0;
        for (int r = 0; r < RUNS; r++) {
            auto s1 = high_resolution_clock::now();
            vector<int> v;
            size_t cap = v.capacity();
            int j = 0;
            for (int i = 0; i < n; i++) {
                v.push_back(rand());
                if (v.capacity() != cap) { cap = v.capacity(); j++; }
            }
            auto e1 = high_resolution_clock::now();
            tPlain += duration<double, micro>(e1 - s1).count();
            jumps = j;
            auto s2 = high_resolution_clock::now();
            vector<int> w;
            w.reserve(n);
            for (int i = 0; i < n; i++) w.push_back(rand());
            auto e2 = high_resolution_clock::now();
            tReserve += duration<double, micro>(e2 - s2).count();
        }
        cout << fixed << setprecision(2) << left << setw(10) << n
        << setw(24) << tPlain / RUNS << setw(28) << tReserve / RUNS
        << jumps << endl;
    }
}
}
// Challenge 1
namespace c1 {
void bubbleSort(int A[], int n) {
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - 1 - i; j++)
            if (A[j] > A[j + 1]) { int t = A[j]; A[j] = A[j + 1]; A[j + 1] = t; }
}
void run() {
    srand(42);
    int sizes[] = {1000, 2000, 5000, 10000, 50000, 100000};
    cout << left << setw(10) << "n" << setw(20) << "Bubble sort (ms)"
    << setw(20) << "std::sort (ms)" << "Ratio" << endl;
    for (int n : sizes) {
        int* base = new int[n];
        for (int i = 0; i < n; i++) base[i] = rand();
        int* copy1 = new int[n];
        int* copy2 = new int[n];
        copy(base, base + n, copy1);
        copy(base, base + n, copy2);
        auto s1 = high_resolution_clock::now();
        bubbleSort(copy1, n);
        auto e1 = high_resolution_clock::now();
        double tBubble = duration<double, milli>(e1 - s1).count();
        auto s2 = high_resolution_clock::now();
        sort(copy2, copy2 + n);
        auto e2 = high_resolution_clock::now();
        double tStd = duration<double, milli>(e2 - s2).count();
        cout << fixed << setprecision(3) << left << setw(10) << n
        << setw(20) << tBubble << setw(20) << tStd
        << setprecision(1) << tBubble / tStd << endl;
        delete[] base; delete[] copy1; delete[] copy2;
    }
}
}
// Challenge 2
namespace c2 {
typedef vector<vector<int> > Matrix;
Matrix makeRandom(int n) {
    Matrix M(n, vector<int>(n));
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            M[i][j] = rand() % 10;
    return M;
}
void multiply(const Matrix& X, const Matrix& Y, Matrix& Z, int n) {
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) {
            int sum = 0;
            for (int k = 0; k < n; k++)
                sum += X[i][k] * Y[k][j];
            Z[i][j] = sum;
        }
}
void run() {
    srand(42);
    int sizes[] = {50, 100, 200, 400, 800};
    cout << left << setw(8) << "n" << setw(20) << "Multiply (ms)"
    << setw(20) << "Approx. bytes" << "Time ratio vs previous" << endl;
    double prev = 0;
    for (int n : sizes) {
        Matrix X = makeRandom(n), Y = makeRandom(n);
        Matrix Z(n, vector<int>(n, 0));
        auto start = high_resolution_clock::now();
        multiply(X, Y, Z, n);
        auto stop = high_resolution_clock::now();
        double ms = duration<double, milli>(stop - start).count();
        cout << fixed << setprecision(3) << left << setw(8) << n
        << setw(20) << ms << setw(20) << 3LL * n * n * sizeof(int);
        if (prev > 0) cout << setprecision(2) << ms / prev << "x";
        else cout << "-";
        cout << endl;
        prev = ms;
    }
}
}
// Challenge 3
namespace c3 {
long long fibRecursive(int n) {
    if (n <= 1) return n;
    return fibRecursive(n - 1) + fibRecursive(n - 2);
}
unsigned long long fibIterative(int n) {
    unsigned long long a = 0, b = 1;
    for (int i = 0; i < n; i++) {
        unsigned long long next = a + b;
        a = b;
        b = next;
    }
    return a;
}
void run() {
    cout << "--- Recursive (naive) ---" << endl;
    cout << left << setw(10) << "n" << setw(16) << "Time (ms)" << "Ratio vs previous" << endl;
    int rn[] = {20, 25, 30, 35};
    double prev = 0;
    for (int n : rn) {
        auto s = high_resolution_clock::now();
        volatile long long r = fibRecursive(n);
        auto e = high_resolution_clock::now();
        (void)r;
        double ms = duration<double, milli>(e - s).count();
        cout << fixed << setprecision(3) << left << setw(10) << n << setw(16) << ms;
        if (prev > 0) cout << setprecision(1) << ms / prev << "x";
        cout << endl;
        prev = ms;
    }
    cout << "\n--- Iterative ---" << endl;
    cout << left << setw(12) << "n" << "Time (us)" << endl;
    int in[] = {20, 35, 1000, 1000000};
    for (int n : in) {
        auto s = high_resolution_clock::now();
        volatile unsigned long long r = fibIterative(n);
        auto e = high_resolution_clock::now();
        (void)r;
        cout << fixed << setprecision(3) << left << setw(12) << n
        << duration<double, micro>(e - s).count() << endl;
    }
}
}
// Challenge 4
namespace c4 {
struct Node {
    int data;
    Node* next;
};
int searchArray(const int* A, int n, int key) {
    for (int i = 0; i < n; i++) if (A[i] == key) return i;
    return -1;
}
int searchVector(const vector<int>& v, int key) {
    for (size_t i = 0; i < v.size(); i++) if (v[i] == key) return (int)i;
    return -1;
}
int searchList(Node* head, int key) {
    int idx = 0;
    for (Node* p = head; p != nullptr; p = p->next, idx++) if (p->data == key) return idx;
    return -1;
}
void run() {
    const int n = 100000;
    const int key = n - 10;
    volatile int sink;
    auto s = high_resolution_clock::now();
    vector<int> v;
    for (int i = 0; i < n; i++) v.push_back(i);
    auto e = high_resolution_clock::now();
    double vIns = duration<double, micro>(e - s).count();
    s = high_resolution_clock::now();
    sink = searchVector(v, key);
    e = high_resolution_clock::now();
    double vSearch = duration<double, micro>(e - s).count();
    size_t vBytes = v.capacity() * sizeof(int);
    s = high_resolution_clock::now();
    int* arr = new int[n];
    for (int i = 0; i < n; i++) arr[i] = i;
    e = high_resolution_clock::now();
    double aIns = duration<double, micro>(e - s).count();
    s = high_resolution_clock::now();
    sink = searchArray(arr, n, key);
    e = high_resolution_clock::now();
    double aSearch = duration<double, micro>(e - s).count();
    size_t aBytes = n * sizeof(int);
    s = high_resolution_clock::now();
    Node* head = nullptr;
    Node* tail = nullptr;
    for (int i = 0; i < n; i++) {
        Node* node = new Node{i, nullptr};
        if (head == nullptr) head = tail = node;
        else { tail->next = node; tail = node; }
    }
    e = high_resolution_clock::now();
    double lIns = duration<double, micro>(e - s).count();
    s = high_resolution_clock::now();
    sink = searchList(head, key);
    e = high_resolution_clock::now();
    double lSearch = duration<double, micro>(e - s).count();
    size_t lBytes = n * sizeof(Node);
    (void)sink;
    cout << "Node size: " << sizeof(Node) << " bytes" << endl << endl;
    cout << left << setw(14) << "Structure" << setw(16) << "Insert (us)"
    << setw(16) << "Search (us)" << setw(16) << "Bytes/element" << "Total bytes" << endl;
    cout << fixed << setprecision(1);
    cout << left << setw(14) << "vector"      << setw(16) << vIns << setw(16) << vSearch
    << setw(16) << (double)vBytes / n << vBytes << endl;
    cout << left << setw(14) << "raw array"   << setw(16) << aIns << setw(16) << aSearch
    << setw(16) << (double)aBytes / n << aBytes << endl;
    cout << left << setw(14) << "linked list" << setw(16) << lIns << setw(16) << lSearch
    << setw(16) << (double)lBytes / n << lBytes << endl;
    delete[] arr;
    while (head != nullptr) {
        Node* next = head->next;
        delete head;
        head = next;
    }
}
}
// Menu
int main(int argc, char* argv[]) {
    string choice;
    if (argc > 1) {
        choice = argv[1];
    } else {
        cout << "CS111 Week 3 Lab - choose an experiment:\n"
        "  1   Exercise 1\n"
        "  2   Exercise 2\n"
        "  3   Exercise 3\n"
        "  4   Exercise 4\n"
        "  5   Exercise 5\n"
        "  c1  Challenge 1\n"
        "  c2  Challenge 2\n"
        "  c3  Challenge 3\n"
        "  c4  Challenge 4\n"
        "  all  Run everything\n"
        "> ";
        cin >> choice;
    }
    bool ran = false;
    auto hdr = [](const char* t) { cout << "\n===== " << t << " =====\n"; };
    bool all = (choice == "all");
    if (all || choice == "1") { hdr("Exercise 1"); ex1::run(); ran = true; }
    if (all || choice == "2") { hdr("Exercise 2"); ex2::run(); ran = true; }
    if (all || choice == "3") { hdr("Exercise 3"); ex3::run(); ran = true; }
    if (all || choice == "4") { hdr("Exercise 4"); ex4::run(); ran = true; }
    if (all || choice == "5") { hdr("Exercise 5 "); ex5::run(); ran = true; }
    if (all || choice == "c1") { hdr("Challenge 1"); c1::run(); ran = true; }
    if (all || choice == "c2") { hdr("Challenge 2"); c2::run(); ran = true; }
    if (all || choice == "c3") { hdr("Challenge 3"); c3::run(); ran = true; }
    if (all || choice == "c4") { hdr("Challenge 4"); c4::run(); ran = true; }
    if (!ran) cout << "Unknown choice: " << choice << endl;
    return 0;
}
