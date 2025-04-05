# pae_nsee_01
Repositorio Destinado para Alunos da PAE_NSEE Projetos Espaciais

#include <SFML/Graphics.hpp>
#include <cmath>
#include <vector>

// Física
struct Point {
    float x, y;
};

int main() {
    // Parâmetros físicos
    const float g = 9.81f;
    const float dt = 0.05f;
    const float mass = 11.0f;
    const float Cd = 0.75f;
    const float rho = 1.225f;
    const float radius = 0.0625f;
    const float A = M_PI * radius * radius;

    // Condições iniciais
    float h = 1168.0f;
    float v0_y = sqrt(2 * g * h);
    float v0 = v0_y / sqrt(2); // sin(45) = 1/sqrt(2)
    float vx = v0, vy = v0;
    float x = 0, y = 0;
    
    std::vector<Point> trajectory;

    // Simulação
    for (int i = 0; i < 10000; ++i) {
        if (y < 0) break;

        float v = sqrt(vx * vx + vy * vy);
        float drag_force = 0.5f * Cd * rho * A * v * v;
        float drag_accel = drag_force / mass;

        float ax = -drag_accel * (vx / v);
        float ay = -g - drag_accel * (vy / v);

        vx += ax * dt;
        vy += ay * dt;
        x += vx * dt;
        y += vy * dt;

        if (y < 0) y = 0;

        trajectory.push_back({x, y});
    }

    // Configurações da janela
    sf::RenderWindow window(sf::VideoMode(800, 600), "Trajetória do Foguete");
    window.setFramerateLimit(60);

    // Conversão de unidades para tela
    auto toScreen = [](Point p) {
        float scale = 0.1f;  // escala para caber na tela
        return sf::Vector2f(p.x * scale, 600 - p.y * scale);  // invertendo Y
    };

    size_t frame = 0;

    while (window.isOpen()) {
        sf::Event e;
        while (window.pollEvent(e)) {
            if (e.type == sf::Event::Closed) window.close();
        }

        window.clear(sf::Color::Black);

        // Desenha trajetória até o frame atual
        for (size_t i = 0; i < std::min(frame, trajectory.size()); ++i) {
            sf::CircleShape point(2);
            point.setPosition(toScreen(trajectory[i]));
            point.setFillColor(sf::Color::Cyan);
            window.draw(point);
        }

        if (frame < trajectory.size()) frame++;

        window.display();
    }

    return 0;
}

