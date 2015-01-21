---
title: "SQLiteException: no such table - in Android"
tags: 
  - Android
  - SQLite
  - Exceptions
---

I was just struggeling for a while trying to help with an Android app. I have almost no specific Android experience, but as the app is pretty basic, I thought - can't be that hard. The beginning seemed pretty easy and straight forward, until we stumbled upon this exception:

    android.database.sqlite.SQLiteException: no such table: <TABLE> (code 1): , while compiling: SELECT <column list> FROM <TABLE>
         at android.database.sqlite.SQLiteConnection.nativePrepareStatement(Native Method)
         at android.database.sqlite.SQLiteConnection.acquirePreparedStatement(SQLiteConnection.java:889)
         at android.database.sqlite.SQLiteConnection.prepare(SQLiteConnection.java:500)
         at android.database.sqlite.SQLiteSession.prepare(SQLiteSession.java:588)
         at android.database.sqlite.SQLiteProgram.<init>(SQLiteProgram.java:58)
         at android.database.sqlite.SQLiteQuery.<init>(SQLiteQuery.java:37)
         at android.database.sqlite.SQLiteDirectCursorDriver.query(SQLiteDirectCursorDriver.java:44)
         at android.database.sqlite.SQLiteDatabase.rawQueryWithFactory(SQLiteDatabase.java:1316)
         at android.database.sqlite.SQLiteDatabase.rawQuery(SQLiteDatabase.java:1255)
         at ...

Nothing about this made any sense, as the table existed from the very beginning of the DB, so we tried different things:

- uninstalling app
- cleaning cache / data
- changing database version number
- forcing to copy the database even if it already existed

(If you have the same exception you can still try one of those points above.)

But as I said, nothing worked, until we started to wonder wether the database actually exists. We couldn't find it on the phone - but this could have been also just because we weren't looking on the right place. So we were checking one more time in the Android Studio folders and that's when we realised, **the names differed**.

## Solution:

The file in the folder didn't have the extension which was used in the code, so **the names were different**. However we didn't get any exception about the file not loaded / not copied or similar, just that the table is missing.