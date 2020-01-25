using System;
using NUnit.Framework;
using FluentAssertions;

namespace BookLibrary
{
    public class BookLibraryTestsTask
    {
        public virtual IBookLibrary CreateBookLibrary() //этот метод удалять нельзя, без него не будет работать
        {
            return new BookLibrary(); //меняется на разные реализации BookLibrary при запуске в системе проверки заданий
        }

        public int GetRandomInt()
        {
            var random = new Random();
            return random.Next(10);
        }

        public int GetRandomInt(int min)
        {
            var random = new Random();
            return random.Next(min, 10);
        }

        //Пример теста. Должен упасть на реализации IncorrectBookLibraryAlwaysFails
        [Test]
        public void SimpleTest()
        {
            var bookLibrary = CreateBookLibrary();
            var id = bookLibrary.AddBook(new Book("Книга1"));
            var book = bookLibrary.GetBookById(id);
            Assert.That(book.Book.Title, Is.EqualTo("Книга1"));
        }

        [Test] //Пустое имя должно приводить к ошибке
        public void EmptyNameTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            Assert.Throws<ArgumentNullException>(delegate { bookLibrary.Enqueue(bookID, ""); });
        }

        [Test] //Отсутствующее имя должно приводить к ошибке 
        public void NullNameTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            Assert.Throws<ArgumentNullException>(delegate { bookLibrary.Enqueue(bookID, null); });
        }

        [Test] //Имена должны быть чувствительны к регистру
        public void CaseSensitiveNamesTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            Assert.DoesNotThrow(delegate { bookLibrary.Enqueue(bookID, "User"); });
        }

        [Test] //Имя не должно изменяться при его записи в очередь
        public void NameInQueueCorrectTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            var userName = "user" + GetRandomInt();
            bookLibrary.Enqueue(bookID, userName);
            foreach (var name in (bookLibrary.GetBookById(bookID)).Queue)
            {
                Assert.That(name, Is.EqualTo(userName));
            }
        }

        [Test] //Имя со специальными символами не должно изменяться при его записи в очередь
        public void NewLineNameTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            bookLibrary.Enqueue(bookID, Environment.NewLine);
            var book = bookLibrary.GetBookById(bookID);
            foreach (var i in book.Queue)
            {
                Assert.That(i, Is.EqualTo(Environment.NewLine));
            }
        }

        [Test] //Нельзя встать в очередь за несуществующей книгой
        public void BookIDDontExistTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookID = Guid.NewGuid();
            Assert.Throws<BookLibraryException>(delegate { bookLibrary.Enqueue(bookID, "user"); });
        }

        [Test] //Взявший книгу пользователь не должен вставать за ней в очередь
        public void TakeBookTwiceTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            Assert.Throws<BookLibraryException>(delegate { bookLibrary.Enqueue(bookID, "user"); });
        }

        [Test] //Пользователь, стоящий в очереди за книгой не должен второй раз попадать в очередь
        public void GetInQueueTwiceTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            bookLibrary.Enqueue(bookID, "user1");
            Assert.Throws<BookLibraryException>(delegate { bookLibrary.Enqueue(bookID, "user1"); });
        }

        [Test] //Пользователь не должен вставать в очередь за книгой, если она не взята
        public void FreeBookTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            Assert.Throws<BookLibraryException>(delegate { bookLibrary.Enqueue(bookID, "user"); });
        }

        [Test] //Пользователь должен встать в очередь за книгой, если она взята
        public void NotFreeBookTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            Assert.That((bookLibrary.GetBookById(bookID)).Queue.Count == 0);
            Assert.DoesNotThrow(delegate { bookLibrary.Enqueue(bookID, "user1"); });
            Assert.That((bookLibrary.GetBookById(bookID)).Queue.Count == 1);
        }

        [Test] //Очередь за книгой вмещает несколько человек
        public void SeveralMenInQueueTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            var usersInQueue = GetRandomInt(2);
            for (int i = 1; i <= usersInQueue; i++)
            {
                bookLibrary.Enqueue(bookID, "user" + i);
            }
            var book = bookLibrary.GetBookById(bookID);
            Assert.That(book.Queue.Count == usersInQueue);
        }

        [Test] //Пользователь, стоящий первым в очереди, может взять книгу как только она освободиться
        //Заодно получилась проверка на неизменность очереди
        public void FirstInQueueGetBookWhenReturnedTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            var usersInQueue = GetRandomInt(2);
            for (int i = 1; i <= usersInQueue; i++)
            {
                bookLibrary.Enqueue(bookID, "user" + i);
            }
            bookLibrary.ReturnBook(bookID);
            for (int i = 1; i <= usersInQueue; i++)
            {
                Assert.DoesNotThrow(delegate { bookLibrary.CheckoutBook(bookID, "user" + i); });
                var book = bookLibrary.GetBookById(bookID);
                Assert.That(book.Queue.Count == usersInQueue - i);
                bookLibrary.ReturnBook(bookID);
            }            
        }

        [Test] //Пользователь, не являющийся первым в очереди, не может брать освободившуюся книгу
        public void SecondInQueueDontGetBookWhenReturnedTest()
        {
            var bookLibrary = CreateBookLibrary();
            var bookName = "Book" + GetRandomInt();
            var bookID = bookLibrary.AddBook(new Book(bookName));
            bookLibrary.CheckoutBook(bookID, "user");
            var usersInQueue = GetRandomInt(2);
            for (int i = 1; i <= usersInQueue; i++)
            {
                bookLibrary.Enqueue(bookID, "user" + i);
            }
            bookLibrary.ReturnBook(bookID);
            Assert.Throws<BookLibraryException>(delegate { bookLibrary.CheckoutBook(bookID, "user2"); });
        }
    }
}
