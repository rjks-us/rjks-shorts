# Scoreboard Library 

The Scoreboard Libary allows you to create custom scoreboards without an refresh bug, it is served with packets.

## Implementing

```java
public class Board {

    public void show(Player player) {
    
        Scoreboard scoreboard = new Scoreboard();
        ScoreboardObjective obj = scoreboard.registerObjective("board", IScoreboardCriteria.b);
        obj.setDisplayName("Title goes here");

        PacketPlayOutScoreboardObjective createPacket = new PacketPlayOutScoreboardObjective(obj, 0);
        PacketPlayOutScoreboardDisplayObjective display = new PacketPlayOutScoreboardDisplayObjective(1, obj);
        PacketPlayOutScoreboardObjective removePacket = new PacketPlayOutScoreboardObjective(obj, 1);

        sendPacket(removePacket, player);
        sendPacket(createPacket, player);
        sendPacket(display, player);

        ArrayList<String> lines = new ArrayList<>();

        lines.add("This");
        lines.add("is");
        lines.add("the");
        lines.add("new");
        lines.add("board");
        lines.add("§dcolors");
        lines.add("are");
        lines.add("also");
        lines.add("possible");
        lines.add("yay");

        lines.forEach(element -> {
            ScoreboardScore score = new ScoreboardScore(scoreboard, obj, element;

            score.setScore(lines.size() - lines.indexOf(element));
            PacketPlayOutScoreboardScore packetPlayOutScoreboardScore = new PacketPlayOutScoreboardScore(score);
            sendPacket(packetPlayOutScoreboardScore, player);
        });
        
    }
    
    private void sendPacket(Packet packet, Player p) {
        ((CraftPlayer)p).getHandle().playerConnection.sendPacket(packet);
    }
}

```

## Imports

```java

/**
 * Packet imports may variate due of different versions
 * */

import net.minecraft.server.v1_8_R3.*;
import org.bukkit.craftbukkit.v1_8_R3.entity.CraftPlayer;
import org.bukkit.entity.Player;
```
