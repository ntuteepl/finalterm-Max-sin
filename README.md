#include <iostream>
#include <cmath>
#include <string>
#include <vector>
#include <stdexcept>             //一個標頭檔，提供一些標準的例外類別，用於處理程式中的異常情況。
using namespace std;

/////////////////////////////////////////////////////////////////////////////////////////////////
// 通用角色模板類別
template <typename KeyType>
class Character {
protected:
    KeyType name;                 // 鍵值（例如名字）
    int level;                    // 等級
    int exp;                      // 經驗值
    int power;                    // 力量
    int knowledge;                // 知識
    int luck;                     // 幸運

public:
    // 構造函數
    Character(KeyType n, int lv, int po, int kn, int lu)
        : name(n), level(lv), exp(0), power(po), knowledge(kn), luck(lu) {}

    // 純虛擬函數：發動角色功能
    virtual void useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) = 0;

    // 打印角色資訊
    virtual void print() {
        cout << name << ": Level " << level
             << ", Power: " << power
             << ", Knowledge: " << knowledge
             << ", Luck: " << luck << "\n";
    }

    // 獲取角色名字
    KeyType getName() {
        return name;
    }
};
////////////////////////////////////////////////////////////////////////////////////////////////////
#ifndef WARRIOR_H
#define WARRIOR_H

// 戰士類別
template <typename KeyType>
class Warrior : public Character<KeyType> {
public:
    Warrior(KeyType n, int lv = 0); // 構造函數
    void useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) override; // 發動角色功能
    void print() override; // 打印角色資訊
};

#endif

template <typename KeyType>
Warrior<KeyType>::Warrior(KeyType n, int lv)
    : Character<KeyType>(n, lv, 30, 0, 0) {}

template <typename KeyType>
void Warrior<KeyType>::useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) {
    monsterHealth -= 30; // 戰士造成30點傷害
    cout << "Warrior " << this->name << " 對怪物造成了 30 點傷害。\n";
}

template <typename KeyType>
void Warrior<KeyType>::print() {
    cout << "Warrior ";
    Character<KeyType>::print();
}

/////////////////////////////////////////////////////////////////////////////////////////////////////////
#ifndef WIZARD_H
#define WIZARD_H

// 法師類別
template <typename KeyType>
class Wizard : public Character<KeyType> {
public:
    Wizard(KeyType n, int lv = 0); // 構造函數
    void useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) override; // 發動角色功能
    void print() override; // 打印角色資訊
};

#endif

template <typename KeyType>
Wizard<KeyType>::Wizard(KeyType n, int lv)
    : Character<KeyType>(n, lv, 0, 30, 0) {}

template <typename KeyType>
void Wizard<KeyType>::useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) {
    teamHealth += 30; // 法師回復30點血量
    if (teamHealth > 100) teamHealth = 100; // 確保隊伍總血量不超過100
    cout << "Wizard " << this->name << " 回復了 30 點血量。\n";
}

template <typename KeyType>
void Wizard<KeyType>::print() {
    cout << "Wizard ";
    Character<KeyType>::print();
}

///////////////////////////////////////////////////////////////////////////////////////////////////////////

#ifndef RIDER_H
#define RIDER_H

// 騎兵類別
template <typename KeyType>
class Rider : public Character<KeyType> {
public:
    Rider(KeyType n, int lv = 0); // 構造函數
    void useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) override; // 發動角色功能
    void print() override; // 打印角色資訊
};

#endif

template <typename KeyType>
Rider<KeyType>::Rider(KeyType n, int lv)
    : Character<KeyType>(n, lv, 0, 0, 30) {}

template <typename KeyType>
void Rider<KeyType>::useAbility(int& teamHealth, int& monsterHealth, bool& blockSpecial) {
    blockSpecial = true; // 騎兵擋住怪物的必殺技
    cout << "Rider " << this->name << " 擋住了怪物的必殺技。\n";
}

template <typename KeyType>
void Rider<KeyType>::print() {
    cout << "Rider ";
    Character<KeyType>::print();
}

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// 自定義例外類別
class MyException : public runtime_error {
public:
    MyException(const string& msg = "") : runtime_error(msg) {}
};

// 隊伍類別
template <typename KeyType>
class Team {
private:
    vector<Character<KeyType>*> member;
    int teamHealth;

public:
    Team() : teamHealth(100) {}
    ~Team() {
        while (this->member.size() > 0) {
            delete this->member.back();
            this->member.pop_back();
        }
    }

    void addWarrior(KeyType name, int lv) throw (MyException) {
        if (member.size() < 3) {
            Warrior<KeyType>* wPtr = new Warrior<KeyType>(name, lv);
            this->member.push_back(wPtr);
        } else {
            throw MyException("Team is full!");
        }
    }

    void addWizard(KeyType name, int lv) throw (MyException) {
        if (member.size() < 3) {
            Wizard<KeyType>* wPtr = new Wizard<KeyType>(name, lv);
            this->member.push_back(wPtr);
        } else {
            throw MyException("Team is full!");
        }
    }

    void addRider(KeyType name, int lv) throw (MyException) {
        if (member.size() < 3) {
            Rider<KeyType>* rPtr = new Rider<KeyType>(name, lv);
            this->member.push_back(rPtr);
        } else {
            throw MyException("Team is full!");
        }
    }

    void useCharacterAbility(int index, int& monsterHealth, bool& blockSpecial) throw (MyException) {
        if (index < 0 || index >= this->member.size()) {
            throw MyException("Invalid character index!");
        }
        this->member[index]->useAbility(this->teamHealth, monsterHealth, blockSpecial);
    }

    void monsterAttack(int round) {
        int damage = (round == 3) ? 40 : 10; // 第三回合使用必殺技
        cout << "怪物對隊伍造成了 " << damage << " 點傷害。\n";
        this->teamHealth -= damage;
    }

    int getTeamHealth() {
        return this->teamHealth;
    }

    void printAllMembers() {
        for (int i = 0; i < this->member.size(); i++) {
            this->member[i]->print();
        }
    }
};

//////////////////////////////////////////////////////////////////////////////////////////////////////////////////

int main() {
    Team<string> team;
    string name1, name2, name3;
    int level1, level2, level3;

    // 輸入三個角色
    cout << "輸入第一個角色 (戰士) 的名字和等級: ";
    cin >> name1 >> level1;
    try {
        team.addWarrior(name1, level1);
        cout << "戰士 " << name1 << " 已添加到隊伍中。\n";
    } catch (MyException& e) {
        cout << e.what() << endl;
    }

    cout << "輸入第二個角色 (法師) 的名字和等級: ";
    cin >> name2 >> level2;
    try {
        team.addWizard(name2, level2);
        cout << "法師 " << name2 << " 已添加到隊伍中。\n";
    } catch (MyException& e) {
        cout << e.what() << endl;
    }

    cout << "輸入第三個角色 (騎兵) 的名字和等級: ";
    cin >> name3 >> level3;
    try {
        team.addRider(name3, level3);
        cout << "騎兵 " << name3 << " 已添加到隊伍中。\n";
    } catch (MyException& e) {
        cout << e.what() << endl;
    }

    // 開始打五隻怪物
    for (int i = 0; i < 5; i++) {
        int monsterHealth = 60;
        bool blockSpecial = false;

        cout << "第 " << (i + 1) << " 隻怪物出現了！\n";

        for (int round = 1; monsterHealth > 0 && team.getTeamHealth() > 0; round++) {
            int choice;
            cout << "選擇角色 (1: Warrior, 2: Wizard, 3: Rider): ";
            cin >> choice;

            try {
				team.useCharacterAbility(choice - 1, monsterHealth, blockSpecial);
            } catch (MyException& e) {
                cout << e.what() << endl;
                continue;
            }

            if (monsterHealth <= 0) {
                cout << "怪物被打敗了！\n";
                break;
            }

            if (round == 3 && blockSpecial) {
                cout << "騎兵擋住了怪物的必殺技！\n";
                blockSpecial = false;
            } else {
                team.monsterAttack(round);
            }

            if (team.getTeamHealth() <= 0) {
                cout << "隊伍總血量歸零，挑戰失敗！\n";
                return 0;
            }
        }

        if (team.getTeamHealth() <= 0) {
            cout << "隊伍總血量歸零，挑戰失敗！\n";
            return 0;
        }
    }

    cout << "打敗五隻怪物，excellent！\n";
    team.printAllMembers();

    return 0;
}
            [![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/_v8RbUGg)
