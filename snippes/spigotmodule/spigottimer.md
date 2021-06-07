# Spigot Module Library 

Spigot Timer gives you the ability to create timers, where you can start, stop or restart repeating timers

## How to

```java
public class TimerTask extends SpigotTimer {

    public TimerTask(long numb1, long numb2, Plugin instance) {
        super(numb1, numb2, instance);
    }

    @Override
    public void execute() {
        System.out.println("Running :)");

        super.execute();
    }
}
```

## Implementing

```java
public class Main {

    ...

    private TimerTask timer;

    public void implementing() {
    
        timer = new TimerTask(1, 1, Main.getInstance());
        timer.start();
        
    }
    
}
```

## Module Class
```java
public class SpigotTimer {

    private int timer;
    private long numb1, numb2;
    private boolean running = false;

    private Plugin instance;

    public SpigotTimer(long numb1, long numb2, Plugin instance) {
        this.numb1 = numb1;
        this.numb2 = numb2;
        this.instance = instance;
    }

    public void execute() {

    }

    public void start() {
        if (!running) {
            running = true;
            timer = Bukkit.getScheduler().scheduleSyncRepeatingTask(instance, new Runnable() {
                @Override
                public void run() {
                    execute();
                }
            }, numb1, numb2);
        }
    }

    public void stop() {
        if (running && (timer != 0)) {
            Bukkit.getScheduler().cancelTask(timer);
            running = false;
        }
    }

    public boolean isRunning() {
        return running;
    }
}
```

## Imports

```java
import org.bukkit.Bukkit;
import org.bukkit.plugin.Plugin;
```
