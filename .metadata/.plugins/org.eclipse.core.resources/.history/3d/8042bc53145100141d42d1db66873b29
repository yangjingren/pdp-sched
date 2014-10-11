import java.io.File;
import java.util.Scanner;

public class Project {
  public float globalTime, lastEventTime;

  public Project(){
    this.globalTime = 0.0f;
    this.lastEventTime = -1.0f;
  }

  class ThreadInfo {
    public int id, requiredTime, priority;
    public float arrivalTime;

    public ThreadInfo(int id, int requiredTime, int priority, float arrivalTime){
      this.id = id;
      this.requiredTime = requiredTime;
      this.priority = priority;
      this.arrivalTime = arrivalTime;
    }
  }

  public void setLastEvent(float time){
    synchronized(this){
      if(lastEventTime < time)
        lastEventTime = time;
    }
  }

  public void advanceGlobalTime(float time){
    synchronized(this){
      float next = (int)(globalTime + 1.0);
      if(time < next && time > 0)
        globalTime = time;
      else
        globalTime = next;
    }
  }

  public ThreadInfo readNext(Scanner scanner){
    if (scanner.hasNext()){
      float arrival = scanner.nextFloat();
      int id = scanner.nextInt();
      int required = scanner.nextInt();
      int priority = scanner.nextInt();

      return new ThreadInfo(id, required, priority, arrival);
    } else {
      return new ThreadInfo(0, 0, 0, 0);
    }
  }

  public class MyThread extends Thread {
    public ThreadInfo info;
    public Scheduler scheduler;
    public Project project;

    public MyThread(ThreadInfo info, Scheduler scheduler, Project project){
      this.info = info;
      this.scheduler = scheduler;
      this.project = project;
    }

    public void run(){
      int timeRemaining = info.requiredTime;
      float scheduledTime;

      project.setLastEvent(info.arrivalTime);

      scheduledTime = scheduler.scheduleme(info.arrivalTime, info.id, timeRemaining, info.priority);

      while (timeRemaining > 0){
        project.setLastEvent(scheduledTime);
        System.out.printf("%.2f - %.2f: T%d\n", scheduledTime, scheduledTime + 1.0, info.id);

        while(project.globalTime < scheduledTime + 1.0) {
          Thread.yield();
        }

        timeRemaining -= 1;
        scheduledTime = scheduler.scheduleme(project.globalTime, info.id, timeRemaining, info.priority);
      }
    }
  }

  public void runTests(int type, String filename) {
    try {
      Scanner scanner = new Scanner(new File(filename));
      Scheduler scheduler = new Scheduler(type);

      ThreadInfo info = readNext(scanner);

      if(info.arrivalTime < 0)
        return;

      MyThread thread = new MyThread(info, scheduler, this);
      thread.start();

      while (lastEventTime < info.arrivalTime){
        Thread.yield();
      }

      info = readNext(scanner);

      while((globalTime - lastEventTime) < 50){
        advanceGlobalTime(info.arrivalTime);

        if (globalTime == info.arrivalTime) {
          thread = new MyThread(info, scheduler, this);
          thread.start();

          while (lastEventTime < info.arrivalTime){
            Thread.yield();
          }

          info = readNext(scanner);
        } else {
          int loopCounter = 0;
          while ((lastEventTime < globalTime) && (loopCounter < 100000)){
            loopCounter++;
            Thread.yield();
          }
        }
      }
    } catch (Exception e) {
    }
  }

  public static void main(String[] args) {
    if (args.length != 2) {
      System.out.println("Not enough parameters specified.  Usage: a.out <scheduler_type> <input_file>");
      System.out.println("  Scheduler type: 0 - First Come, First Served (Non-preemptive)");
      System.out.println("  Scheduler type: 1 - Shortest Remaining Time First (Preemptive)");
      System.out.println("  Scheduler type: 2 - Priority-based Scheduler (Preemptive)");
      System.out.println("  Scheduler type: 3 - Multi-Level Feedback Queue w/ Aging (Preemptive)");
    } else {
      Project project = new Project();
      project.runTests(Integer.parseInt(args[0]), args[1]);
      System.out.println("Finished");
    }
  }
}
