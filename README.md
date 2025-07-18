import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
import java.util.Timer;
import java.util.TimerTask;

public class GoalBuddy {
    private static final DateTimeFormatter TIME_FORMATTER = DateTimeFormatter.ofPattern("h:mm a");
    private static final Scanner inputReader = new Scanner(System.in);
    private static final List<PersonalGoal> myGoals = new ArrayList<>();

    public static void main(String[] args) {
        greetUser();
        
        while (true) {
            showMainMenu();
            
            try {
                int userChoice = Integer.parseInt(inputReader.nextLine());
                
                switch (userChoice) {
                    case 1:
                        createNewGoal();
                        break;
                    case 2:
                        showMyGoals();
                        break;
                    case 3:
                        sayGoodbye();
                        System.exit(0);
                    default:
                        System.out.println("Oops! That's not a valid option. Let's try again.");
                }
            } catch (NumberFormatException e) {
                System.out.println("Please enter just the number (1, 2, or 3).");
            }
        }
    }

    private static void greetUser() {
        System.out.println("\n🌟 Welcome to Daily Goal Buddy! 🌟");
        System.out.println("Let's make today your most productive day yet!");
    }

    private static void showMainMenu() {
        System.out.println("\nWhat would you like to do?");
        System.out.println("1. Add a new personal goal");
        System.out.println("2. See all my current goals");
        System.out.println("3. Exit (I'm done for now)");
        System.out.print("Your choice: ");
    }

    private static void createNewGoal() {
        System.out.println("\n✨ Let's add something awesome to your day! ✨");
        
        System.out.print("What would you like to accomplish? (e.g., 'Morning yoga', 'Read chapter 5'): ");
        String goalDescription = inputReader.nextLine().trim();
        
        if (goalDescription.isEmpty()) {
            System.out.println("Hmm, every goal needs a name! Try again.");
            return;
        }

        LocalTime reminderTime = askForReminderTime();
        if (reminderTime == null) return;

        PersonalGoal newGoal = new PersonalGoal(goalDescription, reminderTime);
        myGoals.add(newGoal);
        setupReminder(newGoal);
        
        System.out.println("\n✅ Got it! I'll remind you to '" + goalDescription + 
                          "' at " + newGoal.getFormattedTime() + ".");
        System.out.println("You've got this! 💪");
    }

    private static LocalTime askForReminderTime() {
        while (true) {
            System.out.print("When should I remind you? (e.g., 6:00 AM, 8:30 PM, or 20:30): ");
            String timeInput = inputReader.nextLine().trim();
            
            if (timeInput.equalsIgnoreCase("cancel")) {
                System.out.println("Okay, let's skip this for now.");
                return null;
            }

            try {
                // Try to parse with AM/PM (with or without space)
                timeInput = timeInput.toUpperCase();
                if (timeInput.contains("AM") || timeInput.contains("PM")) {
                    timeInput = timeInput.replace("AM", " AM").replace("PM", " PM").trim();
                    return LocalTime.parse(timeInput, DateTimeFormatter.ofPattern("h:mm a"));
                } 
                // Try 24-hour format
                else {
                    // Add colon if missing (e.g. "830" -> "8:30")
                    if (!timeInput.contains(":")) {
                        if (timeInput.length() == 3) {
                            timeInput = timeInput.substring(0, 1) + ":" + timeInput.substring(1);
                        } else if (timeInput.length() == 4) {
                            timeInput = timeInput.substring(0, 2) + ":" + timeInput.substring(2);
                        }
                    }
                    return LocalTime.parse(timeInput, DateTimeFormatter.ofPattern("H:mm"));
                }
            } catch (DateTimeParseException e) {
                System.out.println("I didn't understand that time. Please try formats like:");
                System.out.println("- 6:00 AM or 6:00AM");
                System.out.println("- 8:30 PM or 8:30PM");
                System.out.println("- 20:30 (24-hour format)");
                System.out.println("Or type 'cancel' to stop adding this goal.");
            }
        }
    }

    private static void setupReminder(PersonalGoal goal) {
        Timer timer = new Timer();
        LocalTime now = LocalTime.now();
        LocalTime reminderTime = goal.getReminderTime();
        
        long delay;
        
        if (reminderTime.isAfter(now)) {
            delay = java.time.Duration.between(now, reminderTime).toMillis();
        } else {
            // If time passed, schedule for next day
            delay = java.time.Duration.between(now, reminderTime.plusHours(24)).toMillis();
        }
        
        // Add safety check for negative delay
        if (delay < 0) {
            delay = 0;
        }
        
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("\n🔔 *Ding!* Time for your goal!");
                System.out.println("👉 " + goal.getDescription());
                System.out.println("This was scheduled for " + goal.getFormattedTime());
                System.out.println("Make yourself proud! 🌈\n");
                timer.cancel();
            }
        }, delay);
    }

    private static void showMyGoals() {
        if (myGoals.isEmpty()) {
            System.out.println("\nYou haven't set any goals yet. Let's add some!");
            return;
        }
        
        System.out.println("\nHere's what you're working on today:");
        for (int i = 0; i < myGoals.size(); i++) {
            PersonalGoal goal = myGoals.get(i);
            System.out.printf("%d. %s at %s%n", 
                            i + 1, 
                            goal.getDescription(), 
                            goal.getFormattedTime());
        }
        System.out.println("\nLooking great! Keep up the good work! ✨");
    }

    private static void sayGoodbye() {
        System.out.println("\nThanks for using Daily Goal Buddy!");
        System.out.println("Remember: Progress, not perfection. See you tomorrow!");
    }

    private static class PersonalGoal {
        private final String description;
        private final LocalTime reminderTime;

        public PersonalGoal(String description, LocalTime reminderTime) {
            this.description = description;
            this.reminderTime = reminderTime;
        }

        public String getDescription() {
            return description;
        }

        public LocalTime getReminderTime() {
            return reminderTime;
        }

        public String getFormattedTime() {
            return reminderTime.format(TIME_FORMATTER);
        }
    }
}
