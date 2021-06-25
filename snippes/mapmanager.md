# MapManager Library 

The MapManager Libary allows you to create, load, unload custom Minecraft Maps.

#### This requires the SpigotModule Class found [here](https://github.com/rjks-us/rjks-shorts/blob/main/snippes/spigotmodule/spigotmodule.md)

## Implementing

```java
public class Map {

    private String ingameName, configName, author, description, directory;
    private Material icon;
    private boolean loaded;
    private HashMap<String, Object> property = new HashMap<>();
    private HashMap<String, Location> locations = new HashMap<>();

    private Plugin plugin;

    private File configFile, loadedFile;

    public Map(Plugin plugin, String ingameName, String directory, String configName) {
        this.ingameName = ingameName;
        this.configName = configName;
        this.directory = directory;

        this.plugin = plugin;
        this.loaded = false;

        //Check if World is already in plugin directory
        for (World world : Bukkit.getWorlds()) {
            if (world.getName().equalsIgnoreCase(this.configName)) this.loaded = true;
        }

        this.configFile = new File(directory + this.configName);
        this.loadedFile = new File("plugins/../" + this.configName);
    }

    public boolean loadMap() throws Exception {
        if(!isLoaded()) {
            if(!loadedFile.exists()) {
                loadedFile.mkdirs();
                FileUtils.copyDirectory(configFile, loadedFile);
                World world = new WorldCreator(this.configName).createWorld();
                setLoaded(true);
                return true;
            }
        }
        return false;
    }

    public boolean unloadMap() throws Exception {
        if (isLoaded()) {
            for (World world : Bukkit.getWorlds()) {
                if(world.getName().equalsIgnoreCase(configName)) {
                    for (Player all : world.getPlayers()) {
                        all.teleport(Bukkit.getWorlds().get(0).getSpawnLocation());
                    }

                    Bukkit.unloadWorld(world, true);
                    FileUtils.deleteDirectory(this.loadedFile);
                    setLoaded(false);
                    return true;
                }
            }
        }
        return false;
    }

    public void addLocation(String name, Location location) {
        locations.put(name, location);
    }

    public void removeLocation(String name) {
        locations.remove(name);
    }

    public Location getLocation(String name) {
        return locations.get(name);
    }

    public Location getRandomLocation() {
        if (locations.size() == 0) return null;
        return locations.get(new Random().nextInt(locations.size()));
    }

    public void setProperty(String key, Object value) {
        property.put(key, value);
    }

    public Object getProperty(String key) {
        return property.get(key);
    }

    public boolean hasProperty(String key) {
        return (property.get(key) != null);
    }

    public boolean teleportPlayerToWorld(Player player) {
        if (isLoaded()) {
            player.teleport(Bukkit.getWorld(this.configName).getSpawnLocation());
            return true;
        }
        return false;
    }

    public boolean isLoaded() {
        return loaded;
    }

    public String getIngameName() {
        return ingameName;
    }

    public String getDescription() {
        return description;
    }

    public Plugin getPlugin() {
        return plugin;
    }

    public String getDirectory() {
        return directory;
    }

    public File getConfigFile() {
        return configFile;
    }

    public File getLoadedFile() {
        return loadedFile;
    }

    public HashMap<String, Object> getProperty() {
        return property;
    }

    public Material getIcon() {
        return icon;
    }

    public String getAuthor() {
        return author;
    }

    public String getConfigName() {
        return configName;
    }

    public void setLoaded(boolean loaded) {
        this.loaded = loaded;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public void setIcon(Material icon) {
        this.icon = icon;
    }

    public HashMap<String, Location> getLocations() {
        return locations;
    }
}
```

```java
public class MapManager extends SpigotModule {

    private String directory;

    private File mapDirectory;
    private Requirement requirement;
    private ArrayList<Map> mapCache = new ArrayList<>();

    public MapManager(Plugin plugin, String directory, String name, ModuleType type, boolean autoCreate) {
        super(plugin, directory, name, type, autoCreate);
    }

    public Map createMap(String ingameName, String configName) throws Exception {
        if (!new File("plugins/" + getPlugin().getName() + "/maps/" + configName).exists()) return null;

        String path = configName + ".";
        getConfig().set(path + "ingame", ingameName);
        getConfig().set(path + "author", "NO_AUTHOR_PROVIDED");
        getConfig().set(path + "description", "NO_DESCRIPTION_PROVIDED");
        getConfig().set(path + "icon", "RECORD_3");

        save();

        Map map = new Map(getPlugin(), ingameName, directory, configName);
        mapCache.add(map);
        return map;
    }

    /**
     * This has to be called as the default constructor
     * */
    public void initMapManager(Requirement requirement, String directory) {
        this.mapDirectory = new File(directory);
        this.requirement = requirement;
        this.directory = directory;
    }

    public void loadMaps() {
        int amount = 0, setup = 0;
        for (String map : getConfig().getConfigurationSection("").getKeys(false)) {
            Map createdMap = new Map(getPlugin(), getConfig().getString(map + ".ingame"), directory, map);
            createdMap.setAuthor(getConfig().getString(map + ".author"));
            createdMap.setDescription(getConfig().getString(map + ".description"));
            createdMap.setIcon(Material.valueOf(getConfig().getString(map + ".icon")));

            //Properties
            try {
                for (String properties : getConfig().getConfigurationSection(map + ".property").getKeys(false)) {
                    createdMap.setProperty(properties, getConfig().getString(map + ".property." + properties));
                }
            } catch (Exception e) {}

            //Location Collection
            try {
                for (String collection : getConfig().getConfigurationSection(map + ".locationcollection").getKeys(false)) {
                    Location location = new Location(Bukkit.getWorld(createdMap.getConfigName()), 0, 0, 0);
                    location.setX(getConfig().getDouble(map + ".locationcollection." + collection + ".x"));
                    location.setY(getConfig().getDouble(map + ".locationcollection." + collection + ".y"));
                    location.setZ(getConfig().getDouble(map + ".locationcollection." + collection + ".z"));
                    location.setYaw((float) getConfig().getDouble(map + ".locationcollection." + collection + ".yaw"));
                    location.setPitch((float) getConfig().getDouble(map + ".locationcollection." + collection + ".pitch"));

                    createdMap.addLocation(collection, location);
                }
            } catch (Exception e) {}

            //Locations
            try {
                for (String collection : getConfig().getConfigurationSection(map + ".location").getKeys(false)) {
                    Location location = new Location(Bukkit.getWorld(createdMap.getConfigName()), 0, 0, 0);
                    location.setX(getConfig().getDouble(map + ".location." + collection + ".x"));
                    location.setY(getConfig().getDouble(map + ".location." + collection + ".y"));
                    location.setZ(getConfig().getDouble(map + ".location." + collection + ".z"));
                    location.setYaw((float) getConfig().getDouble(map + ".location." + collection + ".yaw"));
                    location.setPitch((float) getConfig().getDouble(map + ".location." + collection + ".pitch"));

                    createdMap.addLocation(collection, location);
                }
            } catch (Exception e) {}

            mapCache.add(createdMap);
            amount++;
            if (mapIsSetUp(createdMap)) setup++;
        }
        getPlugin().getLogger().log(Level.INFO, "[MAP] Loaded " + amount + " maps, " + setup + " are meeting all requirements");
    }

    public boolean mapIsSetUp(Map map) {
        if (this.requirement == null) return true;

        for (String requirementRequirement : this.requirement.getPropertyRequirements()) {
            if (!map.hasProperty(requirementRequirement)) {
                return false;
            }
        }
        for (String requirementRequirement : this.requirement.getLocationRequirements()) {
            boolean state = false;
            for(String name : map.getLocations().keySet()){
                if (name.startsWith(requirementRequirement)) {
                    state = true;
                }
            }
            if (!state) return false;
        }

        for (String requirementRequirement : this.requirement.getLocationCollectionRequirements()) {
            boolean state = false;
            for(String name : map.getLocations().keySet()){
                if (name.startsWith(requirementRequirement)) {
                    state = true;
                }
            }
            if (!state) return false;
        }
        return true;
    }

    /**
     * Property
     * */

    public void addPropertyToMap(Map map, String name, Object object) throws Exception {
        map.setProperty(name, object);

        String path = map.getConfigName() + ".property.";
        getConfig().set(path + "." + name, object);
        save();
    }

    /**
     * Normal Location
     * */

    public void addLocationToMap(Map map, String name, Location location) throws Exception {
        map.addLocation(name, location);

        String path = map.getConfigName() + ".location.";
        getConfig().set(path + "." + name + ".x", Double.valueOf(location.getX()));
        getConfig().set(path + "." + name + ".y", Double.valueOf(location.getY()));
        getConfig().set(path + "." + name + ".z", Double.valueOf(location.getZ()));
        getConfig().set(path + "." + name + ".yaw", Float.valueOf(location.getYaw()));
        getConfig().set(path + "." + name + ".pitch", Float.valueOf(location.getPitch()));
        getConfig().set(path + "." + name + ".world", map.getConfigFile().getName());

        save();
    }

    /**
     * Spawn Collection
     * */

    public void addLocationToCollection(Map map, String collection, Location location) throws Exception {
        int amount = getAmountOfSpawnCollection(map, collection);

        String path = map.getConfigName() + ".locationcollection.";
        getConfig().set(path + "." + collection + "-" + amount + ".x", Double.valueOf(location.getX()));
        getConfig().set(path + "." + collection + "-" + amount + ".y", Double.valueOf(location.getY()));
        getConfig().set(path + "." + collection + "-" + amount + ".z", Double.valueOf(location.getZ()));
        getConfig().set(path + "." + collection + "-" + amount + ".yaw", Float.valueOf(location.getYaw()));
        getConfig().set(path + "." + collection + "-" + amount + ".pitch", Float.valueOf(location.getPitch()));
        getConfig().set(path + "." + collection + "-" + amount + ".world", map.getConfigFile().getName());

        save();
    }

    private int getAmountOfSpawnCollection(Map map, String name) {
        int i = 0;
        try {
            for (String elements : getConfig().getConfigurationSection(map.getConfigName() + ".locationcollection").getKeys(false)) {
                if (elements.startsWith(name)) {
                    i++;
                }
            }
        } catch (Exception e) {}
        return i;
    }

    public Map getMapByName(String name) {
        for (Map map1 : mapCache) {
            if (map1.getConfigName().equalsIgnoreCase(name) || map1.getIngameName().equalsIgnoreCase(name)) {
                return map1;
            }
        }
        return null;
    }

    public void createMapDirectory() throws Exception {
        if (mapDirectory.exists()) mapDirectory.mkdirs();
    }

    public void disable() throws Exception {
        int amount = 0;
        for (Map map : mapCache) {
            map.unloadMap();
            amount++;
        }
        System.out.println("Successfully unloaded " + amount + " Maps");
    }

    public static class Requirement {

        public ArrayList<String> propertyRequirements = new ArrayList<>();
        public ArrayList<String> locationRequirements = new ArrayList<>();
        public ArrayList<String> locationCollectionRequirements = new ArrayList<>();

        public Requirement() {}

        public void addPropertyRequirement(String object) {
            propertyRequirements.add(object);
        }

        public void removePropertyRequirements(String object) {
            propertyRequirements.remove(object);
        }

        public void addLocationRequirement(String object) {
            locationRequirements.add(object);
        }

        public void removeLocationRequirements(String object) {
            locationRequirements.remove(object);
        }

        public void addLocationCollectionRequirement(String object) {
            locationCollectionRequirements.add(object);
        }

        public void removeLocationCollectionRequirements(String object) {
            locationCollectionRequirements.remove(object);
        }

        public ArrayList<String> getLocationRequirements() {
            return locationRequirements;
        }

        public ArrayList<String> getPropertyRequirements() {
            return propertyRequirements;
        }

        public ArrayList<String> getLocationCollectionRequirements() {
            return locationCollectionRequirements;
        }
    }
}

```

## Imports

```java

import org.bukkit.Bukkit;
import org.bukkit.Location;
import org.bukkit.Material;
import org.bukkit.plugin.Plugin;
import us.rjks.module.ModuleType;
import us.rjks.module.SpigotModule;
import org.apache.commons.io.FileUtils;

import java.io.File;
import java.util.ArrayList;
import java.util.logging.Level;
import java.util.HashMap;
import java.util.Random;
```
