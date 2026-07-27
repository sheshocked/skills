---
name: room-database
description: 
category: android
tags: [room-database]
---

## When to Use
Use this skill when implementing local data persistence with Room: entities, DAOs, migrations, Flow queries, or building offline-first data layers.

## Core Components
- **Entity**: Data class representing a database table
- **Dao**: Interface with database operations
- **Database**: Abstract class holding the database connection
- **TypeConverter**: Converts complex types to/from storable primitives

## Workflow
1. Define entities with @Entity
2. Create DAOs with @Dao and query methods
3. Build database with @Database annotation
4. Set up TypeConverters for Date, JSON, etc.
5. Use Flow return types for reactive queries

## Key Patterns
```kotlin
// Entity
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Long,
    @ColumnInfo(name = "display_name") val name: String,
    val email: String,
    @ColumnInfo(defaultValue = "0") val isActive: Boolean
)

// Dao with Flow
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE isActive = 1 ORDER BY display_name")
    fun getActiveUsers(): Flow<List<User>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: User)

    @Delete
    suspend fun delete(user: User)

    @Query("DELETE FROM users")
    suspend fun deleteAll()
}

// TypeConverter
class DateConverter {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? = value?.let { Date(it) }
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? = date?.time
}

// Migration
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE users ADD COLUMN phone TEXT DEFAULT ''")
    }
}
```

## Pitfalls
- **Main thread queries**: Always use suspend or Flow (never run on Main)
- **Migration missing**: Room won't auto-migrate — always write explicit migrations
- **TypeConverter scope**: Converters are global per database
- **Flow distinctUntilChanged**: Room emits on every write, even if data unchanged
- **Testing**: Use in-memory database (Room.inMemoryDatabaseBuilder)

## Verification
- Run instrumented tests with Room.testing runner
- Verify migration with MigrationTestHelper
- Test Flow emissions with Turbine
- Check query plans with EXPLAIN QUERY PLAN