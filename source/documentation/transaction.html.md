---
title: Transaction - ScalikeJDBC
---

## Transaction

<hr/>
### #readOnly block / session
<hr/>

Executes query in read-only mode.

```scala
val names = DB readOnly { implicit session =>
  sql"select name from emp".map { rs => rs.string("name") }.list.apply()
}

val session = DB.readOnlySession
try {
  val names = sql"select name from emp".map { rs => rs.string("name") }.list.apply()
  // do something
} finally {
  session.close()
}
```

Of course, `update` operations in read-only mode will cause `scala.sql.SQLException`.

```scala
DB readOnly { implicit session =>
  sql"update emp set name = ${name} where id = ${id}".update.apply()
} // will throw scala.sql.SQLException
```

<hr/>
### #autoCommit block / session
<hr/>

Executes query / update in auto-commit mode.

```scala
val count = DB autoCommit { implicit session =>
  sql"update emp set name = ${name} where id = ${id}".update.apply()
}
```

When using autoCommitSession, every operation will be executed in auto-commit mode.

```scala
implicit val session = DB.autoCommitSession
try {
  sql"update emp set name = ${name1} where id = ${id1}".update.apply() // auto-commit
  sql"update emp set name = ${name2} where id = ${id2}".update.apply() // auto-commit
} finally { session.close() }
```

<hr/>
### #localTx block
<hr/>

Executes query / update in block-scoped transactions.

If an Exception was thrown in the block, the transaction will perform rollback automatically.

```scala
val count = DB localTx { implicit session =>
  // --- transcation scope start ---
  sql"update emp set name = ${name1} where id = ${id1}".update.apply()
  sql"update emp set name = ${name2} where id = ${id2}".update.apply()
  // --- transaction scope end ---
}
```

<hr/>
### #withinTx block / session
<hr/>

Executes query / update in already existing transactions.

In this case, all the transactional operations (such as `Tx#begin()`, `Tx#rollback()` or `Tx#commit()`) should be managed by users of ScalikeJDBC.

```scala
val db = DB(conn)
try {
  db.begin()
  val names = db withinTx { implicit session =>
    // if a transaction has not been started, IllegalStateException will be thrown
    sql"select name from emp".map { rs => rs.string("name") }.list.apply()
  }
  db.rollback() // it might throw Exception
} finally { db.close() }

val db = DB(conn)
try {
  db.begin()
  implicit val session = db.withinTxSession()
  val names = sql"select name from emp".map { rs => rs.string("name") }.list.apply()
  db.rollbackIfActive() // it NEVER throws Exception
} finally { db.close() }
```


