# simulador-de-batalha
import random
from time import sleep 
from abc import ABC, abstractmethod


# =================================================================
# CLASSES BASE DOS PERSONAGENS (HERÓIS)
# =================================================================

class RPGCharacter(ABC):
    def __init__(self, name, char_class, level=1, health=100, mana=100, xp=0):
        self.name = name
        self.char_class = char_class
        self.level = level
        self.health = health
        self.mana = mana
        self.xp = xp

    def is_alive(self):
        """Verifica se o personagem está vivo."""
        return self.health > 0

    def show_status(self):
        """Exibe o status atual do personagem."""
        return (f"{self.name} | Classe: {self.char_class} | "
                f"Level: {self.level} | HP: {self.health} | Mana: {self.mana} | XP: {self.xp}")

    @abstractmethod
    def stack(self):
        """Define o atributo principal do personagem (abstrato)."""
        pass


class Warrior(RPGCharacter):
    def __init__(self, name):
        super().__init__(name, char_class="Warrior", health=210, mana=0)
        # Habilidade Especial: Escudo (6 usos)
        self.shield_uses = 6 

    def stack(self):
        return "Força"

    def attack(self, enemy):
        damage = random.randint(35, 60)
        enemy.health -= damage
        return f"{self.name} ataca {enemy.name} com espada causando {damage} de dano!"
    
    def use_shield(self):
        """Consome um uso de escudo."""
        if self.shield_uses > 0:
            self.shield_uses -= 1
            # O efeito de defesa é passivo no loop
            return f"{self.name} levanta o escudo! -1 uso. Usos restantes:{self.shield_uses} "
        else:
            return f"{self.name} tentou usar o escudo, mas está esgotado!" 

class Mage(RPGCharacter):
    def __init__(self, name):
        super().__init__(name, char_class="Mage", health=120, mana=200)
        # Habilidade Especial: Poção de Cura (2 usos)
        self.heal_potions = 2

    def stack(self):
        return "Magia"

    def cast_spell(self, enemy):
        damage = random.randint(40, 75)
        if self.mana >= 20:
            self.mana -= 20
            enemy.health -= damage
            return f"{self.name} lança feitiço em {enemy.name} causando {damage} de dano! Mana restante: {self.mana}"
        return f"{self.name} tentou lançar feitiço, mas está sem mana!"
    
    def use_heal_potion(self, heal_amount=35):
        """Cura o Mago consumindo uma poção."""
        if self.heal_potions > 0:
            self.heal_potions -= 1
            self.health += heal_amount
            return f"{self.name} usa Poção de Cura e recupera {heal_amount} de HP! HP atual: {self.health}"
        else:
            return f"{self.name} tentou se curar, mas não tem mais poções!"


class Archer(RPGCharacter):
    def __init__(self, name):
        super().__init__(name, char_class="Archer", health=135, mana=100)
        # Habilidade Especial: Poção de Dano (3 uso)
        self.damage_potions = 3

    def stack(self):
        return "Agilidade"

    def shoot_arrow(self, enemy):
        damage = random.randint(45, 75,)
        # O buff da poção de dano será aplicado no loop, se ativo
        enemy.health -= damage
        return f"{self.name} dispara flecha em {enemy.name} causando {damage} de dano!"

    def use_damage_potion(self):
        """Ativa um buff de dano para o próximo ataque."""
        if self.damage_potions > 0:
            self.damage_potions -= 1
            # O efeito real será manipulado pelo loop principal
            return f"{self.name} bebe a Poção de Dano! O próximo ataque será mais forte!"
        else:
            return f"{self.name} tentou usar poção, mas não tem mais!"


# =================================================================
# CLASSES BASE DOS INIMIGOS
# =================================================================

class Enemy(ABC):
    def __init__(self, name, level=1, health=50, attack_power=10):
        self.name = name
        self.level = level
        self.health = health
        self.attack_power = attack_power

    def is_alive(self):
        """Verifica se o inimigo está vivo."""
        return self.health > 0     

    def show_status(self):
        """Exibe o status atual do inimigo."""
        return (f"{self.name} | Level: {self.level} | "
                f"HP: {self.health} | Ataque Base: {self.attack_power}")

    @abstractmethod
    def attack(self, player):
        """Define o ataque do inimigo (abstrato)."""
        pass


class Goblin(Enemy):
    def __init__(self, name="Goblin", level=1):
        super().__init__(name, level, health=90, attack_power=20)

    def attack(self, player):
        damage = random.randint(15, 20)
        player.health -= damage
        return f"{self.name} morde {player.name} causando {damage} de dano!"


class Orc(Enemy):
    def __init__(self, name="Orc", level=2):
        super().__init__(name, level, health=110, attack_power=15)

    def attack(self, player):
        damage = random.randint(15, 25)
        player.health -= damage
        return f"{self.name} golpeia {player.name} causando {damage} de dano!"


class Dragon(Enemy):
    def __init__(self, name="Dragão", level=5):
        super().__init__(name, level, health=300, attack_power=45)

    def attack(self, player):
        damage = random.randint(45, 55)
        player.health -= damage
        return f"{self.name} cospe fogo em {player.name} causando {damage} de dano!"


# =================================================================
# SIMULADOR DE BATALHA 
# =================================================================

# Instanciando jogadores e inimigos
players = [Warrior("Samuel"), Mage("Guilherme"), Archer("Abreu")]
enemies = [Goblin(), Orc(), Dragon()]

# Variável de estado para o buff do Arqueiro
archer_is_buffed = False
BUFF_MULTIPLIER = 1.5

print("### INÍCIO DA GRANDE BATALHA ###\n")
sleep(3)

round_count = 1

def player_turn(player, enemy):
    """Executa a lógica de ataque e habilidade do herói."""
    global archer_is_buffed
    
    # 25% de chance de usar Habilidade Especial
    use_ability = random.random() < 0.25 
    ability_used = False
    
    if use_ability:
        if isinstance(player, Warrior) and player.shield_uses > 0:
            print(player.use_shield())
            ability_used = True

        elif isinstance(player, Mage) and player.heal_potions > 0:
            print(player.use_heal_potion())
            ability_used = True

        elif isinstance(player, Archer) and player.damage_potions > 0:
            print(player.use_damage_potion())
            archer_is_buffed = True # Ativa o buff
            ability_used = True
            
    # Ataque Normal (ou buffado, se for o arqueiro)
    if not (ability_used and not isinstance(player, Archer)): # Se não usou habilidade que substitui o ataque
        if isinstance(player, Warrior):
            print(player.attack(enemy))
        elif isinstance(player, Mage):
            print(player.cast_spell(enemy))
        elif isinstance(player, Archer):
            # No cenário real, teríamos que refatorar o método shoot_arrow
            # para checar 'archer_is_buffed', mas mantemos simples por enquanto
            print(player.shoot_arrow(enemy))

    # Desativa o buff (ele só dura um turno)
    archer_is_buffed = False 


# O loop principal da batalha: continua enquanto houver heróis E inimigos
while players and enemies:
    print(f"\n{'='*20} ROUND {round_count} {'='*20}")
    
    # 1. Randomizar os combatentes da rodada
    player_combatente = random.choice(players)
    enemy_combatente = random.choice(enemies)
    
    print(f"Batalha: {player_combatente.name} vs. {enemy_combatente.name}")
    print(f"Status Inicial -> {player_combatente.show_status()}")
    print(f"Status Inicial -> {enemy_combatente.show_status()}")
    sleep(1)

    # 2. Randomizar quem ataca primeiro (0 = Jogador, 1 = Inimigo)
    turn_order = random.randint(0, 1)

    # --- FLUXO DO TURNO ---

    # Turno do Jogador primeiro
    if turn_order == 0:
        print("\n[Ordem]: O Herói ataca primeiro!")
        player_turn(player_combatente, enemy_combatente)
        
        sleep(2)

        # Se o inimigo ainda estiver vivo, ele revida
        if enemy_combatente.is_alive():
            print("\n[Ação]: Resposta do Inimigo")
            print(enemy_combatente.attack(player_combatente))
    
    # Turno do Inimigo primeiro
    else:
        print("\n[Ordem]: O Inimigo ataca primeiro!")
        print(enemy_combatente.attack(player_combatente))
        
        sleep(2)

        # Se o jogador ainda estiver vivo, ele revida
        if player_combatente.is_alive():
            print("\n[Ação]: Resposta do Herói")
            player_turn(player_combatente, enemy_combatente)
    
    sleep(2)
    
    # 3. Verificação e Remoção de Mortos
    
    # Remove o inimigo se não estiver mais vivo
    if not enemy_combatente.is_alive():
        print(f"\n*** VITÓRIA! {enemy_combatente.name} foi derrotado! ***")
        if enemy_combatente in enemies: 
            enemies.remove(enemy_combatente)
    
    # Remove o jogador se não estiver mais vivo
    if not player_combatente.is_alive():
        print(f"\n*** DERROTA! {player_combatente.name} caiu em batalha! ***")
        if player_combatente in players: 
            players.remove(player_combatente)
