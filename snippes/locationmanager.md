# Location Library 

Location Manager allows you to define and load location within your plugin

## Implementing

```java
public class Main {

    ...

    private SpawnManager spawnManager;

    public void implementing() {
    
        spawnManager = new SpawnManager(Main.getPlugin().getDataFolder() + "/", "spawns", ModuleType.YML, true);
        spawnManager.loadFile();
        
    }
    
}

```

## Module Class
```java
public class SpawnManager extends SpigotModule {

    public SpawnManager(String directory, String name, ModuleType type, boolean autoCreate) {
        super(directory, name, type, autoCreate);
    }

    public boolean setSpawn(Location location, String name) {
        try {
            getConfig().set(name + ".world", location.getWorld().getName());

            getConfig().set(name + ".x", location.getX());
            getConfig().set(name + ".y", location.getY());
            getConfig().set(name + ".z", location.getZ());

            getConfig().set(name + ".yaw", location.getYaw());
            getConfig().set(name + ".pitch", location.getPitch());

            save();
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    public Location getLocation(String name) {
        if (!spawnExists(name)) return null;

        Location location = new Location(Bukkit.getWorld(getConfig().getString(name + ".world")), 0, 0, 0);

        location.setX((double) getConfig().getDouble(name + ".x"));
        location.setY((double) getConfig().getDouble(name + ".y"));
        location.setZ((double) getConfig().getDouble(name + ".z"));

        location.setYaw((float) getConfig().getDouble(name + ".yaw"));
        location.setPitch((float) getConfig().getDouble(name + ".pitch"));

        return location;
    }

    public boolean spawnExists(String name) {
        return (getConfig().get(name) != null);
    }
}
```

## Imports

```java
import org.bukkit.Bukkit;
import org.bukkit.Location;

/**
 * Note: You also have to import the SpigotModule Library as parent 
 * */
```
