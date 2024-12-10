package com.mygdx.game;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.Input;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Vector2;

public class Player {
    private Vector2 position;
    private float speed = 200f; // Pixels per second
    private Texture texture;
    private ProjectileManager projectileManager;

    public Player(ProjectileManager projectileManager) {
        this.position = new Vector2(400, 300); // Starting position
        this.texture = new Texture("player.png"); // Replace with your player texture
        this.projectileManager = projectileManager;
    }

    public void update(float delta) {
        // Movement
        if (Gdx.input.isKeyPressed(Input.Keys.W)) position.y += speed * delta;
        if (Gdx.input.isKeyPressed(Input.Keys.S)) position.y -= speed * delta;
        if (Gdx.input.isKeyPressed(Input.Keys.A)) position.x -= speed * delta;
        if (Gdx.input.isKeyPressed(Input.Keys.D)) position.x += speed * delta;

        // Keep player within screen bounds
        position.x = Math.max(0, Math.min(position.x, Gdx.graphics.getWidth() - texture.getWidth()));
        position.y = Math.max(0, Math.min(position.y, Gdx.graphics.getHeight() - texture.getHeight()));

        // Shooting
        if (Gdx.input.isButtonJustPressed(Input.Buttons.LEFT)) {
            Vector2 mousePosition = new Vector2(Gdx.input.getX(), Gdx.graphics.getHeight() - Gdx.input.getY());
            projectileManager.spawnProjectile(position, mousePosition);
        }
    }

    public void render(SpriteBatch batch) {
        batch.draw(texture, position.x, position.y);
    }

    public void dispose() {
        texture.dispose();
    }
}

package com.mygdx.game;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.math.Vector2;

public class Projectile {
    private Vector2 position;
    private Vector2 direction;
    private float speed = 500f; // Pixels per second
    private Texture texture;
    private Rectangle bounds;

    public Projectile(Vector2 position, Vector2 target) {
        this.position = new Vector2(position);
        this.direction = target.sub(position).nor(); // Normalize direction
        this.texture = new Texture("projectile.png"); // Replace with your projectile texture
        this.bounds = new Rectangle(position.x, position.y, texture.getWidth(), texture.getHeight());
    }

    public boolean update(float delta) {
        position.add(direction.x * speed * delta, direction.y * speed * delta);
        bounds.setPosition(position);

        // Destroy projectile if it goes out of screen bounds
        return position.x < 0 || position.y < 0 || position.x > Gdx.graphics.getWidth() || position.y > Gdx.graphics.getHeight();
    }

    public void render(SpriteBatch batch) {
        batch.draw(texture, position.x, position.y);
    }

    public Rectangle getBounds() {
        return bounds;
    }

    public void dispose() {
        texture.dispose();
    }
}

package com.mygdx.game;

import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Vector2;

import java.util.ArrayList;
import java.util.Iterator;

public class ProjectileManager {
    private ArrayList<Projectile> projectiles;

    public ProjectileManager() {
        projectiles = new ArrayList<>();
    }

    public void spawnProjectile(Vector2 position, Vector2 target) {
        projectiles.add(new Projectile(position, target));
    }

    public void update(float delta, ArrayList<Enemy> enemies) {
        Iterator<Projectile> iterator = projectiles.iterator();

        while (iterator.hasNext()) {
            Projectile projectile = iterator.next();
            if (projectile.update(delta)) {
                iterator.remove();
                projectile.dispose(); // Cleanup memory
                continue;
            }

            // Check for collision with enemies
            for (Enemy enemy : enemies) {
                if (enemy.getBounds().overlaps(projectile.getBounds())) {
                    enemy.takeDamage(10);
                    iterator.remove();
                    projectile.dispose();
                    break;
                }
            }
        }
    }

    public void render(SpriteBatch batch) {
        for (Projectile projectile : projectiles) {
            projectile.render(batch);
        }
    }
}

package com.mygdx.game;

import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.math.Vector2;

public class Enemy {
    private Vector2 position;
    private Texture texture;
    private int health = 50;
    private Rectangle bounds;

    public Enemy(Vector2 position) {
        this.position = position;
        this.texture = new Texture("enemy.png"); // Replace with your enemy texture
        this.bounds = new Rectangle(position.x, position.y, texture.getWidth(), texture.getHeight());
    }

    public void takeDamage(int damage) {
        health -= damage;
    }

    public boolean isDead() {
        return health <= 0;
    }

    public Rectangle getBounds() {
        return bounds;
    }

    public void render(SpriteBatch batch) {
        batch.draw(texture, position.x, position.y);
    }

    public void dispose() {
        texture.dispose();
    }
}

package com.mygdx.game;

import com.badlogic.gdx.ApplicationAdapter;
import com.badlogic.gdx.graphics.g2d.SpriteBatch;
import com.badlogic.gdx.utils.Array;

import java.util.ArrayList;

public class MyGdxGame extends ApplicationAdapter {
    private SpriteBatch batch;
    private Player player;
    private ProjectileManager projectileManager;
    private ArrayList<Enemy> enemies;

    @Override
    public void create() {
        batch = new SpriteBatch();
        projectileManager = new ProjectileManager();
        player = new Player(projectileManager);

        enemies = new ArrayList<>();
        // Add some enemies to the game
        enemies.add(new Enemy(new Vector2(200, 400)));
        enemies.add(new Enemy(new Vector2(600, 200)));
    }

    @Override
    public void render() {
        float delta = Gdx.graphics.getDeltaTime();

        // Update game logic
        player.update(delta);
        projectileManager.update(delta, enemies);

        // Remove dead enemies
        enemies.removeIf(Enemy::isDead);

        // Render
        batch.begin();
        player.render(batch);
        projectileManager.render(batch);
        for (Enemy enemy : enemies) {
            enemy.render(batch);
        }
        batch.end();
    }

    @Override
    public void dispose() {
        batch.dispose();
        player.dispose();
        for (Enemy enemy : enemies) {
            enemy.dispose();
        }
    }
}
