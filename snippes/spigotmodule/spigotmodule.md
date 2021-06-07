# Spigot Module Library 

Spigot Module gives you the ability to create or load custom configuration files.

## How to

```java
public class Config extends SpigotModule {

    public Config(String directory, String name, ModuleType type, boolean autoCreate) {
        super(directory, name, type, autoCreate);
    }
    
}

```

## Implementing

```java
public class Main {

    ...

    private Config config;

    public void implementing() {
    
        config = new Config(Main.getPlugin().getDataFolder() + "/", "config", ModuleType.YML, false);
        config.loadTemplate("config.yml");
        config.loadFile();
        
    }
    
}

```

## Module Class
```java
public class SpigotModule {

    private String name, directory;
    private ModuleType type;

    private File dir;

    private File file;
    private YamlConfiguration config;

    private HashMap<String, Object> cache = new HashMap<>();

    public SpigotModule(String directory, String name, ModuleType type, boolean autoCreate) {
        this.name = name;
        this.directory = directory;
        this.type = type;
        this.dir = new File(directory);
        this.file = new File(getDirectory(), getName() + "." + getType().toString().toLowerCase());

        try {
            if (autoCreate) {
                if (!dirExists()) createDirectory();
                if (!fileExists()) createEmptyFile();

                loadFile();
            }
        } catch (Exception exception) {}
    }

    public void createEmptyFile() throws Exception {
        if (!fileExists()) {
            getFile().createNewFile();
        }
    }

    public void createDirectory() throws Exception {
        if (!dir.exists()) {
            dir.mkdirs();
        }
    }

    public void loadFromCache(String name) throws Exception {
        if (!fileExists()) {
            FileUtils.copyInputStreamToFile(Main.getPlugin().getResource(name), new File(getDirectory(), name));
        }
    }

    public void loadFile() {
        config = YamlConfiguration.loadConfiguration(getFile());
    }

    public void loadTemplate(String name) throws Exception {
        if (!dirExists()) createDirectory();
        loadFromCache(name);
    }

    public void reloadFile() throws Exception {
        cache.clear();
        config = YamlConfiguration.loadConfiguration(getFile());
    }

    public void save() throws Exception {
        config.save(getFile());
    }

    public boolean fileExists() {
        return file.exists();
    }

    public boolean dirExists() {
        return dir.exists();
    }

    public File getFile() {
        return file;
    }

    public YamlConfiguration getConfig() {
        return config;
    }

    public String getName() {
        return name;
    }

    public String getDirectory() {
        return directory;
    }

    public ModuleType getType() {
        return type;
    }

    public HashMap<String, Object> getCache() {
        return cache;
    }
}

public enum ModuleType {
    YML,
    JSON
}
```

## Imports

```java
import org.apache.commons.io.FileUtils;
import org.bukkit.configuration.file.YamlConfiguration;
import us.rjks.core.Main;

import java.io.File;
import java.util.HashMap;
```
