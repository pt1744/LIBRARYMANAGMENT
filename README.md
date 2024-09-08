# LIBRARYMANAGMENT

#include <iostream>
#include <string>
#include <vector>
#include <unordered_map>
#include <ctime>

using namespace std;

// Book class
class Book {
public:
    string title;
    string author;
    string bookID;
    int copiesAvailable;

    Book(string t, string a, string i, int c) : title(t), author(a), bookID(i), copiesAvailable(c) {}
};

// Borrower class
class Borrower {
public:
    string name;
    string borrowerID;
    vector<pair<string, time_t>> borrowedBooks;  // Pair of bookID and borrow date
    

  Borrower() : name(""), borrowerID("") {}

    Borrower(string n, string id) : name(n), borrowerID(id) {}

    void borrowBook(const string& bookID, time_t borrowDate) {
        borrowedBooks.push_back(make_pair(bookID, borrowDate));
    }

    void returnBook(const string& bookID) {
        for (auto it = borrowedBooks.begin(); it != borrowedBooks.end(); ++it) {
            if (it->first == bookID) {
                borrowedBooks.erase(it);
                break;
            }
        }
    }

    bool hasOverdueBooks() const {
        time_t now = time(0);
        for (const auto& book : borrowedBooks) {
            double daysBorrowed = difftime(now, book.second) / (60 * 60 * 24);
            if (daysBorrowed > 15) {
                return true;
            }
        }
        return false;
    }

    bool canBorrowMoreBooks() const {
        return borrowedBooks.size() < 4;  // Limit to 4 books
    }
};

// Library class
class Library {
private:
    vector<Book> books;
    unordered_map<string, Borrower> borrowers;

public:
    void addBook(const Book& book) {
        books.push_back(book);
    }

    void listBooks() {
        for (const auto& book : books) {
            cout << "Title: " << book.title << ", Author: " << book.author << ", Book ID: " << book.bookID << ", Copies Available: " << book.copiesAvailable << endl;
        }
    }

    vector<Book*> searchBooksByTitle(const string& title) {
        vector<Book*> foundBooks;
        for (auto& book : books) {
            if (book.title == title) {
                foundBooks.push_back(&book);
            }
        }
        return foundBooks;
    }

    Borrower* searchBorrower(const string& borrowerID) {
        if (borrowers.find(borrowerID) != borrowers.end()) {
            return &borrowers[borrowerID];
        }
        return nullptr;
    }

    void addBorrower(const Borrower& borrower) {
        borrowers[borrower.borrowerID] = borrower;
    }

    void borrowBook(const string& borrowerID, const string& title) {
        Borrower* borrower = searchBorrower(borrowerID);
        if (!borrower) {
            cout << "Borrower not found." << endl;
            return;
        }

        if (!borrower->canBorrowMoreBooks()) {
            cout << "You have already borrowed 4 books. Please return one before borrowing a new one." << endl;
            return;
        }

        if (borrower->hasOverdueBooks()) {
            cout << "You have overdue books. Please return them before borrowing a new one." << endl;
            return;
        }

        vector<Book*> foundBooks = searchBooksByTitle(title);
        if (foundBooks.empty()) {
            cout << "No books found with the title \"" << title << "\"." << endl;
            return;
        } else if (foundBooks.size() == 1) {
            Book* book = foundBooks.front();
            if (book->copiesAvailable > 0) {
                borrower->borrowBook(book->bookID, time(0));
                book->copiesAvailable--;
                cout << borrower->name << " borrowed \"" << book->title << "\"" << endl;
            } else {
                cout << "No copies available for \"" << book->title << "\"" << endl;
            }
        } else {
            cout << "Multiple books found with the title \"" << title << "\". Please enter the book ID:" << endl;
            for (const auto& book : foundBooks) {
                cout << "Book ID: " << book->bookID << ", Author: " << book->author << ", Copies Available: " << book->copiesAvailable << endl;
            }
            string bookID;
            cout << "Enter the Book ID: ";
            cin.ignore();
            getline(cin, bookID);
            for (auto& book : foundBooks) {
                if (book->bookID == bookID) {
                    if (book->copiesAvailable > 0) {
                        borrower->borrowBook(book->bookID, time(0));
                        book->copiesAvailable--;
                        cout << borrower->name << " borrowed \"" << book->title << "\"" << endl;
                    } else {
                        cout << "No copies available for \"" << book->title << "\"" << endl;
                    }
                    return;
                }
            }
            cout << "Invalid Book ID." << endl;
        }
    }

    void returnBook(const string& borrowerID, const string& bookID) {
        Borrower* borrower = searchBorrower(borrowerID);
        if (!borrower) {
            cout << "Borrower not found." << endl;
            return;
        }

        for (auto& book : books) {
            if (book.bookID == bookID) {
                borrower->returnBook(bookID);
                book.copiesAvailable++;
                cout << borrower->name << " returned \"" << book.title << "\"" << endl;
                return;
            }
        }

        cout << "Invalid Book ID." << endl;
    }
};
int main() {
    Library library;
    int option;
    while (true) {
        cout << "\nLibrary Management System\n";
        cout << "1. Add Book\n";
        cout << "2. Add Borrower\n";
        cout << "3. Borrow Book\n";
        cout << "4. Return Book\n";
        cout << "5. List Books\n";
        cout << "6. Exit\n";
        cout << "Enter your option: ";
        cin >> option;

        if (option == 1) {
            string title, author, bookID;
            int copiesAvailable;
            cout << "Enter book title: ";
            cin.ignore();
            getline(cin, title);
            cout << "Enter book author: ";
            getline(cin, author);
            cout << "Enter book ID: ";
            getline(cin, bookID);
            cout << "Enter number of copies available: ";
            cin >> copiesAvailable;

            library.addBook(Book(title, author, bookID, copiesAvailable));
            cout << "Book added successfully.\n";
        } else if (option == 2) {
            string name, borrowerID;
            cout << "Enter borrower name: ";
            cin.ignore();
            getline(cin, name);
            cout << "Enter borrower ID: ";
            getline(cin, borrowerID);

            library.addBorrower(Borrower(name, borrowerID));
            cout << "Borrower added successfully.\n";
        } else if (option == 3) {
            string borrowerID, title;
            cout << "Enter borrower ID: ";
            cin.ignore();
            getline(cin, borrowerID);
            cout << "Enter book title to borrow: ";
            getline(cin, title);

            library.borrowBook(borrowerID, title);
        } else if (option == 4) {
            string borrowerID, bookID;
            cout << "Enter borrower ID: ";
            cin.ignore();
            getline(cin, borrowerID);
            cout << "Enter book ID to return: ";
            getline(cin, bookID);

            library.returnBook(borrowerID, bookID);
        } else if (option == 5) {
            cout << "\nBooks in the library:\n";
            library.listBooks();
        } else if (option == 6) {
            cout << "Exiting the program.\n";
            break;
        } else {
            cout << "Invalid option. Please try again.\n";
        }
    }
    return 0;
}
