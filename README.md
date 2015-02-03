# hello-world-2
Second edition
using System;  
 using Microsoft.SPOT;  
 using Gadgeteer.Modules.GHIElectronics;  
 using GT = Gadgeteer;  
 namespace GadgeteerRoboticArm  
 {  
   public class RoboticArmController  
   {  
     private const int ClawPwmMin = 1500;  
     private const int ClawPwmMax = 2200;  
     private const int ArmPwmMin = 600;  
     private const int ArmPwmMax = 2400;
  
     private const int ClawPositionMax = 100;  
     private const int ClawPositionMin = 0;  
     private const int ArmPositionMax = 100;  
     private const int ArmPositionMin = 40; 
 
     private uint _pwmPulsePeriod;  
     private GT.Interfaces.PWMOutput _clawPwm;  
     private GT.Interfaces.PWMOutput _armPwm; 
 
     public int ClawCurrentPosition { get; private set; }  
     public int ArmCurrentPosition { get; private set; } 
 
     public RoboticArmController(Extender extender, GT.Socket.Pin clawPin, GT.Socket.Pin armPin, uint pwmPulsePeriod)  
     {  
       if (extender == null)  
       {  
         throw new ApplicationException("robotic arm pwm extender not set up correctly");  
       }  
       _pwmPulsePeriod = pwmPulsePeriod;  
       _clawPwm = extender.SetupPWMOutput(clawPin);  
       _armPwm = extender.SetupPWMOutput(armPin);  
       Reset();  
     }
  
     public void Reset()  
     {  
       MoveClaw(ClawPositionMin);  
       MoveArm(ArmPositionMin);  
     } 
 
     public void MoveClaw(int position)  
     {  
       ClawCurrentPosition = Move(_clawPwm, position, ClawPositionMin, ClawPositionMax, ClawPwmMin, ClawPwmMax);  
     }
  
     public void MoveArm(int position)  
     {  
       ArmCurrentPosition = Move(_armPwm, position, ArmPositionMin, ArmPositionMax, ArmPwmMin, ArmPwmMax);  
     } 
 
     private int Move(GT.Interfaces.PWMOutput pwm, int position, int minPosition, int maxPosition, int pwmMin, int pwmMax)  
     {  
       position = position > maxPosition ? maxPosition : position;  
       position = position < minPosition ? minPosition : position;
  
       Servo(pwm, GetPwmPulseValue(pwmMax, pwmMin, position));
  
       return position;  
     } 
 
     private int GetPwmPulseValue(int max, int min, int position)  
     {  
       return (min + (((max - min) / 100) * position)) * 1000;  
     }
  
     private void Servo(GT.Interfaces.PWMOutput pwm, int pwmPulseHighTime)  
     {  
       pwm.SetPulse(_pwmPulsePeriod, (uint)pwmPulseHighTime);  
     }  
   }  
 }  
 {  
   public partial class Program  
   {  
     private RoboticArmController _roboticArmController;  
     private GT.Timer _timer; 
 
     void ProgramStarted()  
     {  
       Debug.Print("Program Started");  
       _roboticArmController = new RoboticArmController(extender, GT.Socket.Pin.Seven, GT.Socket.Pin.Eight, 20000000);
  
       button.ButtonPressed += new Button.ButtonEventHandler(button_ButtonPressed);  
       button.ButtonReleased +=new Button.ButtonEventHandler(button_ButtonReleased);
  
       _timer = new GT.Timer(500);  
       _timer.Tick += new GT.Timer.TickEventHandler(_timer_Tick);      
     }
  
     void _timer_Tick(GT.Timer timer)  
     {  
       _roboticArmController.MoveArm((int)potentiometer.ReadPotentiometerPercentage());  
     }
  
     void button_ButtonPressed(Button sender, Button.ButtonState state)  
     {  
       _roboticArmController.MoveClaw(100);  
     }
  
     void button_ButtonReleased(Button sender, Button.ButtonState state)  
     {  
       _roboticArmController.MoveClaw(0);  
     }  
   }  
 }  
