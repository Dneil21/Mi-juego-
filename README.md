// Screenshot.hpp
#pragma once
#include <SFML/Graphics.hpp>
#include <vector>
#include <string>

class Screenshot {
public:
    Screenshot();
    void capture(const sf::RenderWindow& window);
    void saveToFile(const std::string& filename);
    void loadGallery();
    void displayGallery(sf::RenderWindow& window);
    void deleteScreenshot(int index);

private:
    std::vector<sf::Image> gallery;
    const int MAX_SCREENSHOTS = 60;
};

// Screenshot.cpp
#include "Screenshot.hpp"
#include <iostream>

Screenshot::Screenshot() {
    loadGallery();
}

void Screenshot::capture(const sf::RenderWindow& window) {
    if (gallery.size() >= MAX_SCREENSHOTS) {
        std::cout << "Gallery is full. Please delete some screenshots." << std::endl;
        return;
    }

    sf::Texture texture;
    texture.create(window.getSize().x, window.getSize().y);
    texture.update(window);
    gallery.push_back(texture.copyToImage());

    std::cout << "Screenshot taken. Total: " << gallery.size() << "/" << MAX_SCREENSHOTS << std::endl;
}

void Screenshot::saveToFile(const std::string& filename) {
    if (!gallery.empty()) {
        gallery.back().saveToFile(filename);
        std::cout << "Screenshot saved to " << filename << std::endl;
    }
}

void Screenshot::loadGallery() {
    // Here you would implement loading saved screenshots from files
    // For simplicity, we're not implementing this now
}

void Screenshot::displayGallery(sf::RenderWindow& window) {
    // Here you would implement displaying the gallery
    // This would involve creating thumbnails and handling user interaction
    // For simplicity, we're not implementing this fully now
}

void Screenshot::deleteScreenshot(int index) {
    if (index >= 0 && index < gallery.size()) {
        gallery.erase(gallery.begin() + index);
        std::cout << "Screenshot deleted. Total: " << gallery.size() << "/" << MAX_SCREENSHOTS << std::endl;
    }
}
</antArtif
// UI.hpp
#pragma once
#include <SFML/Graphics.hpp>
#include "Player.hpp"

class UI {
public:
    UI();
    void update(const Player& player, float dt);
    void draw(sf::RenderWindow& window);

private:
    sf::Font font;
    sf::Text playerInfo;
    sf::Text btosTimer;
    sf::RectangleShape btosButton;
    sf::Text btosButtonText;

    void updateBtosTimer(const Player& player);
    std::string formatTime(float seconds);
};

// UI.cpp
#include "UI.hpp"
#include <sstream>
#include <iomanip>

UI::UI() {
    font.loadFromFile("arial.ttf");  // Make sure to have this font file

    playerInfo.setFont(font);
    playerInfo.setCharacterSize(20);
    playerInfo.setFillColor(sf::Color::Black);
    playerInfo.setPosition(10, 10);

    btosTimer.setFont(font);
    btosTimer.setCharacterSize(24);
    btosTimer.setFillColor(sf::Color::Black);
    btosTimer.setPosition(10, 40);

    btosButton.setSize(sf::Vector2f(100, 30));
    btosButton.setFillColor(sf::Color::Green);
    btosButton.setPosition(10, 70);

    btosButtonText.setFont(font);
    btosButtonText.setCharacterSize(16);
    btosButtonText.setFillColor(sf::Color::White);
    btosButtonText.setPosition(15, 75);
}

void UI::update(const Player& player, float dt) {
    std::stringstream ss;
    ss << "Level: " << player.getLevel() << " | XP: " << player.getXP()
       << " | Btos: " << player.getBtos() << " | Coins: " << player.getCoins();
    playerInfo.setString(ss.str());

    updateBtosTimer(player);

    btosButtonText.setString(player.isBtosActive() ? "Stop btos" : "Start btos");
}

void UI::draw(sf::RenderWindow& window) {
    window.draw(playerInfo);
    window.draw(btosTimer);
    window.draw(btosButton);
    window.draw(btosButtonText);
}

void UI::updateBtosTimer(const Player& player) {
    if (player.isBtosActive()) {
        btosTimer.setString(formatTime(player.getBtosTimer()));
        if (player.getBtosTimer() < 600) {  // Less than 10 minutes
            btosTimer.setFillColor(sf::Color::Red);
        } else {
            btosTimer.setFillColor(sf::Color::Black);
        }
    } else {
        btosTimer.setString("");
    }
}

std::string UI::formatTime(float seconds) {
    int minutes = static_cast<int>(seconds) / 60;
    int hours = minutes / 60;
    minutes %= 60;
    int secs = static_cast<int>(seconds) % 60;

    std::stringstream ss;
    ss << std::setfill('0') << std::setw(2) << hours << ":"
       << std::setfill('0') << std::setw(2) << minutes << ":"
       << std::setfill('0') << std::setw(2) << secs;
    return ss.str();
}
// Server.hpp
#pragma once
#include <vector>
#include "Player.hpp"
#include "Food.hpp"

enum class ServerType {
    Normal,
    Farmer
};

class Server {
public:
    Server(ServerType type);
    void update(Player& player, std::vector<Food>& foods, float dt);
    void switchType(ServerType newType);

private:
    ServerType type;
    void updateNormalServer(Player& player, std::vector<Food>& foods, float dt);
    void updateFarmerServer(Player& player, std::vector<Food>& foods, float dt);
};

// Server.cpp
#include "Server.hpp"
#include <random>

Server::Server(ServerType type) : type(type) {}

void Server::update(Player& player, std::vector<Food>& foods, float dt) {
    switch (type) {
        case ServerType::Normal:
            updateNormalServer(player, foods, dt);
            break;
        case ServerType::Farmer:
            updateFarmerServer(player, foods, dt);
            break;
    }
}

void Server::switchType(ServerType newType) {
    type = newType;
}

void Server::updateNormalServer(Player& player, std::vector<Food>& foods, float dt) {
    // Check collision with food
    auto it = foods.begin();
    while (it != foods.end()) {
        if (player.getPosition().x - player.getMass() < it->getPosition().x &&
            player.getPosition().x + player.getMass() > it->getPosition().x &&
            player.getPosition().y - player.getMass() < it->getPosition().y &&
            player.getPosition().y + player.getMass() > it->getPosition().y) {
            player.grow(it->getMass());
            it = foods.erase(it);
        } else {
            ++it;
        }
    }

    // Respawn food
    while (foods.size() < 1000) {
        foods.emplace_back(rand() % 5000, rand() % 5000);
    }
}

void Server::updateFarmerServer(Player& player, std::vector<Food>& foods, float dt) {
    // Similar to normal server, but with different rules for food respawn and growth
    // For example, food might respawn faster or give more mass
    // This is where you'd implement the specific rules for the Farmer server
}
// Player.hpp
#pragma once
#include <SFML/Graphics.hpp>

class Player {
public:
    Player(float x, float y);
    void move(sf::Vector2f target);
    void update(float dt);
    void draw(sf::RenderWindow& window);
    void toggleBtos();
    void addBtos(int amount);
    void grow(float amount);
    sf::Vector2f getPosition() const;

    int getLevel() const { return level; }
    int getXP() const { return xp; }
    int getBtos() const { return btos; }
    int getCoins() const { return coins; }
    float getMass() const { return mass; }
    bool isBtosActive() const { return btosActive; }
    float getBtosTimer() const { return btosTimer; }

private:
    sf::CircleShape shape;
    float mass;
    int level;
    int xp;
    int btos;
    int coins;
    bool btosActive;
    float btosTimer;
    float speed;
};

// Player.cpp
#include "Player.hpp"
#include <cmath>

Player::Player(float x, float y) : mass(1000), level(1), xp(0), btos(0), coins(0),
                                   btosActive(false), btosTimer(0), speed(3) {
    shape.setRadius(std::sqrt(mass / 3.14159f));
    shape.setPosition(x, y);
    shape.setFillColor(sf::Color::Blue);
}

void Player::move(sf::Vector2f target) {
    sf::Vector2f direction = target - shape.getPosition();
    float length = std::sqrt(direction.x * direction.x + direction.y * direction.y);
    if (length != 0) {
        direction /= length;
        shape.move(direction * speed);
    }
}

void Player::update(float dt) {
    if (btosActive) {
        btosTimer -= dt;
        if (btosTimer <= 0) {
            btosActive = false;
            btosTimer = 0;
        }
    }
}

void Player::draw(sf::RenderWindow& window) {
    window.draw(shape);
}

void Player::toggleBtos() {
    if (btos > 0) {
        btosActive = !btosActive;
        if (btosActive && btosTimer == 0) {
            btosTimer = 3600;  // 1 hour in seconds
            btos--;
        }
    }
}

void Player::addBtos(int amount) {
    btos += amount;
    btosTimer += amount * 60;  // 1 bto = 1 minute
}

void Player::grow(float amount) {
    mass += amount;
    shape.setRadius(std::sqrt(mass / 3.14159f));
}

sf::Vector2f Player::getPosition() const {
    return shape.getPosition();
}
// main.cpp
#include "Game.hpp"

int main() {
    Game game;
    game.run();
    return 0;
}

// Game.hpp
#pragma once
#include <SFML/Graphics.hpp>
#include "Player.hpp"
#include "Food.hpp"
#include "Shop.hpp"
#include "Server.hpp"
#include "UI.hpp"
#include "Screenshot.hpp"

class Game {
public:
    Game();
    void run();

private:
    void handleInput();
    void update(float dt);
    void render();

    sf::RenderWindow window;
    Player player;
    std::vector<Food> foods;
    Shop shop;
    Server currentServer;
    UI ui;
    Screenshot screenshot;
    sf::View camera;
    float zoomLevel;
};

// Game.cpp
#include "Game.hpp"

Game::Game() : window(sf::VideoMode(800, 600), "Agar.io Clone"), 
               player(400, 300),
               currentServer(ServerType::Normal) {
    camera.setSize(800, 600);
    camera.setCenter(400, 300);
    window.setView(camera);
    zoomLevel = 1.0f;
}

void Game::run() {
    sf::Clock clock;
    while (window.isOpen()) {
        float dt = clock.restart().asSeconds();
        handleInput();
        update(dt);
        render();
    }
}

void Game::handleInput() {
    sf::Event event;
    while (window.pollEvent(event)) {
        if (event.type == sf::Event::Closed)
            window.close();
        if (event.type == sf::Event::KeyPressed) {
            if (event.key.code == sf::Keyboard::Q)
                player.toggleBtos();
            if (event.key.code == sf::Keyboard::F12)
                screenshot.capture(window);
        }
        if (event.type == sf::Event::MouseWheelScrolled) {
            zoomLevel += event.mouseWheelScroll.delta * 0.1f;
            zoomLevel = std::max(0.5f, std::min(zoomLevel, 2.0f));
            camera.setSize(800 / zoomLevel, 600 / zoomLevel);
        }
    }

    // Player movement
    sf::Vector2i mousePos = sf::Mouse::getPosition(window);
    sf::Vector2f worldPos = window.mapPixelToCoords(mousePos);
    player.move(worldPos);
}

void Game::update(float dt) {
    player.update(dt);
    camera.setCenter(player.getPosition());
    window.setView(camera);

    currentServer.update(player, foods, dt);
    ui.update(player, dt);
}

void Game::render() {
    window.clear(sf::Color::White);

    for (const auto& food : foods) {
        food.draw(window);
    }

    player.draw(window);
    ui.draw(window);

    window.display();
}
