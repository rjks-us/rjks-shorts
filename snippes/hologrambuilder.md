# Scoreboard Library 

The Hologram Libary allows you to create custom holograms, it is served with packets.

## Implementing

```java
public class Main {

    ...

    public void sendHologram(Player player) {

        SpigotHologram holo = new SpigotHologram(Main.getPlugin(), player.getLocation());
        holo.addLine("Test Hologram");
        holo.load();
        holo.showTemporary(player, 30);
        
    }
    
}
```

## Implementing

```java
public class SpigotHologram {

    private HashMap<EntityArmorStand, Location> armorStands = new HashMap<>();
    private List<String> lines = new ArrayList<>();

    private ArrayList<Player> players = new ArrayList<>();

    private Plugin plugin;

    private Location location;
    private double margin = 0.25d;

    private boolean small = false, baseplate = false;

    public SpigotHologram(Plugin plugin, Location location) {
        this.location = location;
        this.plugin = plugin;
    }

    public void addLine(String line) {
        lines.add(line);
    }

    public void removeLine(String line) {
        lines.remove(line);
    }

    public void showTemporary(Player player, int ticks) {
        show(player);
        Bukkit.getScheduler().scheduleSyncDelayedTask(plugin, new Runnable() {
            @Override
            public void run() {
                hide(player);
            }
        }, ticks);
    }

    public void show(Player player) {
        players.add(player);
        armorStands.forEach((entityArmorStand, location1) -> {
            PacketPlayOutSpawnEntityLiving packet = new PacketPlayOutSpawnEntityLiving(entityArmorStand);
            ((CraftPlayer) player).getHandle().playerConnection.sendPacket(packet);
        });
    }

    public void hide(Player player) {
        players.remove(player);
        armorStands.forEach((entityArmorStand, location1) -> {
            PacketPlayOutEntityDestroy packet = new PacketPlayOutEntityDestroy(entityArmorStand.getId());
            ((CraftPlayer) player).getHandle().playerConnection.sendPacket(packet);
        });
    }

    public void showForAll() {
        Bukkit.getOnlinePlayers().forEach(player -> {
            players.add(player);
            armorStands.forEach((entityArmorStand, location1) -> {
                PacketPlayOutSpawnEntityLiving packet = new PacketPlayOutSpawnEntityLiving(entityArmorStand);
                ((CraftPlayer) player).getHandle().playerConnection.sendPacket(packet);
            });
        });
    }

    public void hideForAll() {
        Bukkit.getOnlinePlayers().forEach(player -> {
            players.remove(player);
            armorStands.forEach((entityArmorStand, location1) -> {
                PacketPlayOutEntityDestroy packet = new PacketPlayOutEntityDestroy(entityArmorStand.getId());
                ((CraftPlayer) player).getHandle().playerConnection.sendPacket(packet);
            });
        });
    }

    public void load() {
        armorStands.clear();
        lines.forEach(s -> {
            EntityArmorStand armorStand = new EntityArmorStand(((CraftWorld) this.location.getWorld()).getHandle(), this.location.getX(), this.location.getY(), this.location.getZ());

            armorStand.setSmall(small);
            armorStand.setCustomName(s);
            armorStand.setCustomNameVisible(true);
            armorStand.setInvisible(true);
            armorStand.setGravity(false);
            armorStand.setBasePlate(baseplate);

            armorStands.put(armorStand, location);
            this.location.subtract(0, margin, 0);
        });

        armorStands.forEach((entityArmorStand, location1) -> {
            this.location.add(0, location1.getY() - margin, 0);
        });
    }

    public boolean isShownToPlayer(Player player) {
        return players.contains(player);
    }

    public double getMargin() {
        return margin;
    }

    public List<String> getLines() {
        return lines;
    }

    public HashMap<EntityArmorStand, Location> getArmorStands() {
        return armorStands;
    }

    public Location getLocation() {
        return location;
    }

    public Plugin getPlugin() {
        return plugin;
    }

    public boolean isBaseplate() {
        return baseplate;
    }

    public boolean isSmall() {
        return small;
    }

    public void setBaseplate(boolean baseplate) {
        this.baseplate = baseplate;
    }

    public void setLines(List<String> lines) {
        this.lines = lines;
    }

    public void setLocation(Location location) {
        this.location = location;
    }

    public void setMargin(double margin) {
        this.margin = margin;
    }

    public void setPlugin(Plugin plugin) {
        this.plugin = plugin;
    }

    public void setSmall(boolean small) {
        this.small = small;
    }
}


```

## Imports

```java

/**
 * Packet imports may variate due of different versions
 * */

import net.minecraft.server.v1_8_R3.*;
import org.bukkit.Bukkit;
import org.bukkit.Location;
import org.bukkit.craftbukkit.v1_8_R3.CraftWorld;
import org.bukkit.craftbukkit.v1_8_R3.entity.CraftPlayer;
import org.bukkit.entity.Player;
import org.bukkit.plugin.Plugin;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
```
