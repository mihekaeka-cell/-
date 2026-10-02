```
package interfaces;

public interface Manageable {
    void startWork();  // Запустить / Включить
    void stopWork(); // Остановить / Выключить
    boolean isWorking();      // Статус работы (true/false)
}

ГЛАВНЫЙ ОДИН
package models;

import interfaces.Manageable;

public abstract class FactoryEquipment implements Manageable {
    protected String id, name;
    protected boolean status;
    protected double temperature; // Основной показатель (температура / нагрузка / мощность)

    public FactoryEquipment(String id, String name, boolean status, double temperature) {
        this.id = id; this.name = name; this.status = status; this.temperature = temperature;
    }

    @Override public void startWork() { this.status = true; }
    @Override public void stopWork() { this.status = false; }
    @Override public boolean isWorking() { return status; }

    public abstract String getDetails();

    public String getId() { return id; }
    public String getName() { return name; }
    public double getTemperature() { return temperature; }
    public void setTemperature(double temperature) { this.temperature = temperature; }
}

подштука
package models;

public class ConveyorBelt extends FactoryEquipment {
    private double speed; // Уникальное поле (speed, ramSize, brightness и т.д.)

    public ConveyorBelt(String id, String name, boolean status, double temperature, double speed) {
        super(id, name, status, temperature);
        this.speed = speed;
    }

    public double getSpeed() { return speed; }
    public void setSpeed(double speed) { this.speed = speed; }

    @Override
    public String getDetails() {
        return String.format("%s [ID: %s, Name: %s, Active: %b, Metric: %.1f, Spec: %.1f]",
                getClass().getSimpleName(), id, name, status, temperature, speed);
    }
}

ПРОТОКОЛ
package logic;

import models.FactoryEquipment;
import java.util.function.Consumer;
import java.util.function.Predicate;

public class SafetyProtocol<T extends FactoryEquipment> {
    private String protocolName;
    private Predicate<T> condition;
    private Consumer<T> action;

    public SafetyProtocol(String ruleName, Predicate<T> condition, Consumer<T> action) {
        this.protocolName = ruleName;
        this.condition = condition;
        this.action = action;
    }

    @SuppressWarnings("Не проверенно")
    public void apply(FactoryEquipment item) {
        try {
            T target = (T) item;
            if (condition.test(target)) {
                action.accept(target);
                System.out.println("Правило '" + protocolName + "' сработало для " + item.getName());
            }
        } catch (ClassCastException ignored) {}
    }
}


ЛОГИКА МАНАГЕР

package logic;

import models.FactoryEquipment;
import java.util.*;
import java.util.stream.Stream;

public class FactoryManager {
    private Map<String, List<FactoryEquipment>> groups = new HashMap<>();
    private List<SafetyProtocol<? extends FactoryEquipment>> rules = new ArrayList<>();

    public void addItem(String groupName, FactoryEquipment item) {
        groups.computeIfAbsent(groupName, k -> new ArrayList<>()).add(item);
    }

    public void addRule(SafetyProtocol<?> rule) {
        rules.add(rule);
    }

    public FactoryEquipment getItemById(String id) {
        return getAnalyticsStream()
                .filter(item -> item.getId().equalsIgnoreCase(id))
                .findFirst().orElse(null);
    }

    public void applyAllRules() {
        getAnalyticsStream().forEach(item -> rules.forEach(rule -> rule.apply(item)));
    }

    public Stream<FactoryEquipment> getAnalyticsStream() {
        return groups.values().stream().flatMap(List::stream);
    }
}


МЕЙН

package main;

import logic.SafetyProtocol;
import logic.FactoryManager;
import models.*;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        FactoryManager manager = new FactoryManager();

        while (true) {
            System.out.println("\n=== МЕНЮ УПРАВЛЕНИЯ ===");
            System.out.println("1. Добавить объект");
            System.out.println("2. Изменить статус (Вкл/Выкл)");
            System.out.println("3. Добавить базовое правило");
            System.out.println("4. Добавить индивидуальное правило");
            System.out.println("5. Выполнить все правила");
            System.out.println("6. Выполнить аналитику (Stream API)");
            System.out.println("7. Выход");
            System.out.print("Выбор: ");

            int choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1 -> {
                    System.out.print("Выберите тип оборудования (1 - Робот, 2 - Конвейер, 3 - Пресс): ");
                    int type = sc.nextInt();
                    sc.nextLine(); // очистка буфера послеnextInt()

                    System.out.print("Группа (Цех/Комната/Проект): "); String group = sc.nextLine();
                    System.out.print("ID: "); String id = sc.nextLine();
                    System.out.print("Имя: "); String name = sc.nextLine();
                    System.out.print("Показатель метрики (температура/нагрузка): "); double metric = sc.nextDouble();

                    FactoryEquipment item;

                    // В зависимости от цифры спрашиваем специфичный параметр и создаем нужный объект:
                    switch (type) {
                        case 1 -> {
                            System.out.print("Введите грузоподъемность / яркрость / RAM: ");
                            double spec = sc.nextDouble();
                            item = new RoboticArm(id, name, false, metric, spec);
                        }
                        case 2 -> {
                            System.out.print("Введите скорость / целевую температуру: ");
                            double spec = sc.nextDouble();
                            item = new ConveyorBelt(id, name, false, metric, spec);
                        }
                        default -> {
                            System.out.print("Введите давление / активные соединения: ");
                            double spec = sc.nextDouble();
                            item = new HydraulicPress(id, name, false, metric, spec);
                        }
                    }

                    manager.addItem(group, item);
                    System.out.println("Объект успешно создан и добавлен!");
                }
                case 2 -> {
                    System.out.print("ID: ");
                    FactoryEquipment item = manager.getItemById(sc.nextLine());
                    if (item != null) {
                        if (item.isWorking()) item.stopWork(); else item.startWork();
                        System.out.println("Статус изменен: " + item.getDetails());
                    } else System.out.println("Не найдено!");
                }
                case 3 -> {
                    // Базовый сценарий: сброс показателей у выключенных объектов
                    manager.addRule(new SafetyProtocol<FactoryEquipment>(
                            "Сброс фантомных показателей",
                            item -> !item.isWorking() && item.getTemperature() > 20,
                            item -> item.setTemperature(20)
                    ));
                    System.out.println("Базовое правило добавлено.");
                }
                case 4 -> {
                    manager.addRule(new SafetyProtocol<RoboticArm>(
                            "Критическая перегрузка Аппарата А",
                            // Условие: если работает, первичная метрика > 90 И специфичная > 50
                            a -> a.isWorking() && a.getTemperature() > 85.0,
                            // Действие: выключить
                            RoboticArm::stopWork
                    ));
                    System.out.println("Индивидуальное правило для Типа А добавлено.");
                }
                case 5 -> {
                    manager.applyAllRules();
                    System.out.println("Проверка завершена.");
                }
                case 6 -> {
                    boolean hasDanger = manager.getAnalyticsStream()
                            .anyMatch(item -> item.getTemperature() > 150.0);
                    System.out.println("Есть ли активные объекты с показателем > 150? " + (hasDanger ? "Да" : "Нет"));
                }
                case 7 -> {
                    System.out.println("Завершение работы.");
                    return;
                }
                default -> System.out.println("Неверный ввод!");
            }
        }
    }
}

ШТУКИ
// --- Сценарий 1: Энергосбережение (SmartSocket) ---
manager.addPolicy(new GenericPolicy<SmartSocket>(
        "Энергосбережение",
        socket -> socket.isActive() && socket.getCurrentLoad() > 2000.0,
        SmartSocket::stop
));

// --- Сценарий 2: Спокойный сон (SmartLight) ---
manager.addPolicy(new GenericPolicy<SmartLight>(
        "Спокойный сон",
        light -> light.getBrightness() > 50,
        light -> {
            light.setBrightness(10);
            light.setColor("Warm");
        }
));

// --- Сценарий 3: Эко-режим климата (SmartThermostat) ---
manager.addPolicy(new GenericPolicy<SmartThermostat>(
        "Эко-режим климата",
        term -> term.isActive() && term.getTargetTemperature() > 25.0,
        term -> term.setTargetTemperature(22.0)
));

// --- Сценарий 4: Умное обесточивание (Все устройства) ---
manager.addPolicy(new GenericPolicy<BaseEntity>(
        "Умное обесточивание",
        device -> device.isActive() && device.getMainMetric() == 0, // mainMetric = powerConsumption
        BaseEntity::stop
));

// --- Сценарий 5: Имитация присутствия (SmartLight) ---
manager.addPolicy(new GenericPolicy<SmartLight>(
        "Имитация присутствия",
        light -> !light.isActive(),
        light -> {
            light.start();
            light.setBrightness(100);
        }
));

// --- Сценарий 6: Безопасный обогрев (SmartThermostat) ---
manager.addPolicy(new GenericPolicy<SmartThermostat>(
        "Безопасный обогрев",
        term -> term.getTargetTemperature() < 15.0,
        term -> {
            term.start();
            term.setTargetTemperature(20.0);
        }
));

// --- Сценарий 7: Ночная подсветка (SmartLight) ---
manager.addPolicy(new GenericPolicy<SmartLight>(
        "Ночная подсветка",
        light -> !light.isActive() && light.getName().toLowerCase().contains("коридор"),
        light -> {
            light.start();
            light.setColor("Red");
            light.setBrightness(5);
        }
));

// --- Сценарий 8: Защита ТВ (SmartSocket) ---
manager.addPolicy(new GenericPolicy<SmartSocket>(
        "Защита ТВ",
        socket -> socket.getName().toUpperCase().contains("TV") && socket.getMainMetric() < 10.0,
        SmartSocket::stop
));

АНАЛ

System.out.println("--- ТОП-3 устройства по энергопотреблению ---");
manager.getAnalyticsStream()
        .sorted(Comparator.comparingDouble(BaseEntity::getMainMetric).reversed())
        .limit(3)
        .forEach(d -> System.out.println(d.getDetails()));


double totalPower = manager.getAnalyticsStream()
        .filter(BaseEntity::isActive)
        .mapToDouble(BaseEntity::getMainMetric)
        .sum();

System.out.printf("Суммарное энергопотребление: %.2f Вт\n", totalPower);

List<String> roomsWithOffDevices = manager.getGroupMap().entrySet().stream()
        .filter(entry -> entry.getValue().stream().anyMatch(device -> !device.isActive()))
        .map(Map.Entry::getKey)
        .collect(Collectors.toList());

System.out.println("Комнаты с выключенными устройствами: " + String.join(", ", roomsWithOffDevices));



Map<Boolean, List<BaseEntity>> groupedByStatus = manager.getAnalyticsStream()
        .collect(Collectors.partitioningBy(BaseEntity::isActive));

System.out.println("Включенные устройства (" + groupedByStatus.get(true).size() + " шт.):");
groupedByStatus.get(true).forEach(d -> System.out.println(" - " + d.getName()));

System.out.println("Выключенные устройства (" + groupedByStatus.get(false).size() + " шт.):");
groupedByStatus.get(false).forEach(d -> System.out.println(" - " + d.getName()));




Optional<BaseEntity> minPowerDevice = manager.getAnalyticsStream()
        .filter(BaseEntity::isActive)
        .min(Comparator.comparingDouble(BaseEntity::getMainMetric));

if (minPowerDevice.isPresent()) {
    System.out.println("Минимальное потребление: " + minPowerDevice.get().getDetails());
} else {
    System.out.println("Нет активных устройств.");
}



Map<String, Long> countByClass = manager.getAnalyticsStream()
        .collect(Collectors.groupingBy(
                device -> device.getClass().getSimpleName(),
                Collectors.counting()
        ));

countByClass.forEach((className, count) -> 
        System.out.println(className + ": " + count + " шт."));



boolean hasHighPower = manager.getAnalyticsStream()
        .anyMatch(device -> device.getMainMetric() > 3000.0);

System.out.println("Есть устройство с потреблением > 3000 Вт: " + (hasHighPower ? "Да" : "Нет"));




String activeNames = manager.getAnalyticsStream()
        .filter(BaseEntity::isActive)
        .map(BaseEntity::getName)
        .collect(Collectors.joining(", "));

System.out.println("Работающие устройства: " + (activeNames.isEmpty() ? "Ничего не включено" : activeNames));
