Title: Sorting an ArrayList of Objects
Date: 2013-12-18 16:54
Category: Solutions
Tags: collections, sorting, array, java

Fortunately Java's ``Collection``s library offers a ``sort`` method. You can take an ArrayList of some object and sort
it by some field in the object. There are a few caveats. One is that you can't sort by an ``int`` field. Fortunately you
can just take the ``int``s and convert them to ``Double``s.

```java
Collections.sort(userList, new Comparator<User>() {
    @Override
    public double compare(User u1, User u2) {
        return new Double(u2.getCalcdUptime())
            .compareTo(new Double(u1.getCalcdUptime()));
    }
});
```
The method's signature looks crazy here (reminds me of JS). [Here it is from Oracle's documentation](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#sort(java.util.List, java.util.Comparator)).
