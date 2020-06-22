#Import all libraries needed
import json
import os
import shutil
import time
import board
import busio
import adafruit_mpl115a2
import adafruit_adxl34x
from datetime import datetime

#Wait 5 seconds for the Pi to boot up
time.sleep(10)

#Define directory location to dump json data packets
path = "/tmp/experiment"

#Define communication bus for the sensors
i2c = busio.I2C(board.SCL, board.SDA)      

#Initialize and define varaibles used throughout the code
init_temp_sensor = 0
init_accel = 0
temp_status = "Not working"
accel_status = "Not working"
press_total = 0
temp_total = 0
x_axis_total = 0
y_axis_total = 0
z_axis_total = 0
reboost_constant = 40
count = 1
count_file = open("/home/pi/Documents/count.txt", "r+")
internal_count = int(count_file.read())
num_of_measurements = 290000000

try:
    #Attempt to create the directory
    os.mkdir(path)      
except OSError:
    #Catch exception if creation of directory fails
    print("Directory already existed %s" % path)    
else:
    print("Succesfully created the directory %s" % path)                        


#loop to iterate until communication is established with the sensors
while init_temp_sensor == 0 and init_accel == 0:
   
    try:
        #Attempt to create an object for the temperature sensor
        temp_sensor = adafruit_mpl115a2.MPL115A2(i2c)
        lowest_temp = temp_sensor.temperature
        highest_temp = temp_sensor.temperature
        lowest_press = temp_sensor.pressure
        highest_press = temp_sensor.pressure
        init_temp_sensor = 1
         
           
    except:
        #catch exception if object creation fails
        print ("No temperature sensor found")  
       
    try:
        #Attempt to create an object for the accelerometer
        accelerometer = adafruit_adxl34x.ADXL345(i2c)
        accelerometer.data_rate = adafruit_adxl34x.DataRate.RATE_3200_HZ
        accelerometer.range = adafruit_adxl34x.Range.RANGE_2_G
        highest_x_axis = accelerometer.acceleration[0]
        lowest_x_axis = accelerometer.acceleration[0]
        highest_y_axis = accelerometer.acceleration[1]
        lowest_y_axis = accelerometer.acceleration[1]
        highest_z_axis = accelerometer.acceleration[2]
        lowest_z_axis = accelerometer.acceleration[2]
        init_accel = 1

           
    except:
        #Catch exception if object creation fails
        print ("No accelerometer found")  
       
    time.sleep(1)


#Define function to calculate average values
def calculate_avg (count, total, new):  
    average = (total + new)/count
    return average
   
def minimum_val(new, lowest): #function to calculate the minimum temperature
    if new < lowest:
        lowest = new
    return lowest

def maximum_val(new, highest):    #function to calculate the maximum temperature
    if new > highest:
        highest = new        
    return highest

#Define function to check for orbital reboosts
def orbit_reboost_check (new, avg, highest, lowest):
    if((abs(new)) > ((2*(abs(avg))) + (abs(highest)) + reboost_constant) or (abs(new)) > ((2*(abs(avg))) + (abs(lowest)) + reboost_constant)):
        return ["Reboost detected", "X axis: " + (str(accelerometer.acceleration[0])), "Y axis: " + (str(accelerometer.acceleration[1])), "Z axis: " + (str(accelerometer.acceleration[0]))]
    else:
        return ["No reboost detected"]

while(num_of_measurements > internal_count):
    try:
        new_temp = temp_sensor.temperature
        avg_temp = calculate_avg (count, temp_total, new_temp)
        lowest_temp = minimum_val(new_temp,lowest_temp)
        highest_temp = maximum_val(new_temp,highest_temp)
       
        new_press = temp_sensor.pressure
        avg_press = calculate_avg (count, press_total, new_press)
        lowest_press = minimum_val(new_press, lowest_press)
        highest_press = maximum_val(new_press, highest_press)
       
        print ("The current temperature is: " + (str(new_temp)) + "°C")
        print ("The average temperature is: " + (str(avg_temp)) + "°C")
        print ("The highest temperature recorded is: " + (str(highest_temp))+ "°C")
        print ("The lowesst temperature recorded is: " + (str(lowest_temp))+ "°C")
        print ()
        print ("The current pressure is: " + (str(new_press)) + "hPa")
        print ("The average pressure is: " + (str(avg_press)) + " hPa")
        print ("The highest pressure recorded is: " + (str(highest_press))+ " hPa")
        print ("The lowest pressure recorded is: " + (str(lowest_press))+ " hPa")
        print ()
       
        temp_status = "Working"
        press_total = press_total + new_press
        temp_total = temp_total + new_temp
        if (init_accel == 0):
            count = count + 1
               
    except:
        print("Calculation error with temperature sensor")
       
    try:
        new_x_axis = accelerometer.acceleration[0]
        new_y_axis = accelerometer.acceleration[1]
        new_z_axis = accelerometer.acceleration[2]
        x_axis_avg = calculate_avg(count, x_axis_total, new_x_axis)
        y_axis_avg = calculate_avg(count, y_axis_total, new_z_axis)
        z_axis_avg = calculate_avg(count, z_axis_total, new_z_axis)
        x_reboost_check = orbit_reboost_check(new_x_axis, x_axis_avg, highest_x_axis, lowest_x_axis)
        y_reboost_check = orbit_reboost_check(new_y_axis, y_axis_avg, highest_y_axis, lowest_y_axis)
        z_reboost_check = orbit_reboost_check(new_z_axis, z_axis_avg, highest_z_axis, lowest_z_axis)
        highest_x_axis = maximum_val(new_x_axis, highest_x_axis)
        highest_y_axis = maximum_val(new_y_axis, highest_y_axis)
        highest_z_axis = maximum_val(new_z_axis, highest_z_axis)
        lowest_x_axis = minimum_val(new_x_axis, lowest_x_axis)
        lowest_y_axis = minimum_val(new_y_axis, lowest_y_axis)
        lowest_z_axis = minimum_val(new_z_axis, lowest_z_axis)
       
        print ("The current acceleration is: " + "(" + (str(new_x_axis)) + ", " + (str(new_y_axis)) + ", " + (str(new_z_axis)) + ")")
        print ("The average acceleration is: " + "(" + (str(x_axis_avg)) + ", " + (str(y_axis_avg)) + ", " + (str(z_axis_avg)) + ")")
        print ("The highest acceleration recorded is: " + "(" + (str(highest_x_axis)) + ", " + (str(highest_y_axis)) + ", " + (str(highest_z_axis))+ ")")
        print ("The lowest acceleration recorded is: " + "(" + (str(lowest_x_axis)) + ", " + (str(lowest_y_axis)) + ", " + (str(lowest_z_axis)) + ")")
        print()
       
        accel_status = "Working"
        x_axis_total = x_axis_total + new_x_axis
        y_axis_total = y_axis_total + new_y_axis
        z_axis_total = z_axis_total + new_z_axis
       
        count = count + 1
    except:
        print("Calculation error with accelerometer")
       
    try:
           
        if(init_temp_sensor == 1 and init_accel == 1):
            json_data = {"DaVinci HS" : "Team Toscani", (str(time.time())) : (str(datetime.fromtimestamp(time.time()))), "MPL115A2 status" : "Initialized, " + temp_status, "ADXL345 status" : "Initialized, " + accel_status}
            json_data['Temperature'] = []
            json_data['Temperature'].append({
                "Current temperature" : new_temp,                            
                "Average temperature" : avg_temp,
                "Lowest temperature" : lowest_temp,
                "Highest temperature" : highest_temp
            })
            json_data['Pressure'] = []
            json_data['Pressure'].append({
                "Current pressure" : new_press,
                "Average pressure" : avg_press,
                "Lowest pressure" : lowest_press,
                "Highest pressure" : highest_press
            })
           
            json_data['Acceleration'] = {}
            json_data['Acceleration']['Current acceleration'] = []
            json_data['Acceleration']['Current acceleration'].append({
                "X Axis" : new_x_axis,
                "Y Axis" : new_y_axis,
                "Z Axis" : new_z_axis
            })
            json_data['Acceleration']['Average acceleration'] = []
            json_data['Acceleration']['Average acceleration'].append({
                "X Axis" : x_axis_avg,
                "Y Axis" : y_axis_avg,
                "Z Axis" : z_axis_avg
            })
            json_data['Acceleration']['Lowest acceleration'] = []
            json_data['Acceleration']['Lowest acceleration'].append({
                "X Axis" : lowest_x_axis,
                "Y Axis" : lowest_y_axis,
                "Z Axis" : lowest_z_axis
            })
            json_data['Acceleration']['Highest acceleration'] = []
            json_data['Acceleration']['Highest acceleration'].append({
                "X Axis" : highest_x_axis,
                "Y Axis" : highest_y_axis,
                "Z Axis" : highest_z_axis
            })
           
            json_data['Acceleration']['Reboosts'] = []
           
            if(x_reboost_check[0] == "Reboost detected" or y_reboost_check[0] == "Reboost detected" or z_reboost_check[0] == "Reboost detected"):
                if(x_reboost_check[0] == "Reboost detected"):
                    json_data['Acceleration']['Reboosts'].append(x_reboost_check)
                elif(y_reboost_check[0] == "Reboosts detected"):
                    json_data['Acceleration']['Reboosts'].append(y_reboost_check)
                else:
                    json_data['Acceleration']['Reboosts'].append(z_reboost_check)
            else:
                    json_data['Acceleration']['Reboosts'].append({
                        "X axis: ":"No reboosts have been detected",
                        "Y axis: ":"No reboosts have been detected",
                        "Z axis: ":"No reboosts have been detected",
                    })
           
           
            filepath = (path + '/' + str(time.time()) + '.json')
            json_file = json.dumps(json_data, indent = 4, separators=(',', ': '))
            with open(filepath, 'w') as outfile:
                outfile.write(json_file)
        elif (init_temp_sensor == 1 and init_accel == 0):
            son_data = {"DaVinci HS" : "Team Toscani", (str(time.time())) : (str(datetime.fromtimestamp(time.time()))), "MPL115A2 status" : "Initialized, " + temp_status, "ADXL345 status" : "Initialized, " + accel_status}
            json_data['Temperature'] = []
            json_data['Temperature'].append({
                "Current temperature" : (str(new_temp)),                            
                "Average temperature" : avg_temp,
                "Lowest temperature" : lowest_temp,
                "Highest temperature" : highest_temp
            })
            json_data['Pressure'] = []
            json_data['Pressure'].append({
                "Current pressure" : new_press,
                "Average pressure" : avg_press,
                "Lowest pressure" : lowest_press,
                "Highest pressure" : highest_press
            })
           
            filepath = (path + '/' + str(time.time()) + '.json')
            json_file = json.dumps(json_data, indent = 4, separators=(',', ': '))
            with open(filepath, 'w') as outfile:
                outfile.write(json_file)
               
        elif(init_temp_sensor == 0 and init_accel == 1):
            json_data['Acceleration'] = {}
            json_data['Acceleration']['Current acceleration'] = []
            json_data['Acceleration']['Current acceleration'].append({
                "X Axis" : new_x_axis,
                "Y Axis" : new_y_axis,
                "Z Axis" : new_z_axis
            })
            json_data['Acceleration']['Average acceleration'] = []
            json_data['Acceleration']['Average acceleration'].append({
                "X Axis" : x_axis_avg,
                "Y Axis" : y_axis_avg,
                "Z Axis" : z_axis_avg
            })
            json_data['Acceleration']['Lowest acceleration'] = []
            json_data['Acceleration']['Lowest acceleration'].append({
                "X Axis" : lowest_x_axis,
                "Y Axis" : lowest_y_axis,
                "Z Axis" : lowest_z_axis
            })
            json_data['Acceleration']['Highest acceleration'] = []
            json_data['Acceleration']['Highest acceleration'].append({
                "X Axis" : highest_x_axis,
                "Y Axis" : highest_y_axis,
                "Z Axis" : highest_z_axis
            })
           
            if(x_reboost_check[0] == "Reboost detected" or y_reboost_check[0] == "Reboost detected" or z_reboost_check[0] == "Reboost detected"):
                if(x_reboost_check[0] == "Reboost detected"):
                    json_data['Acceleration']['Reboosts'].append(x_reboost_check)
                elif(y_reboost_check[0] == "Reboosts detected"):
                    json_data['Acceleration']['Reboosts'].append(y_reboost_check)
                else:
                    json_data['Acceleration']['Reboosts'].append(z_reboost_check)
            else:
                    json_data['Acceleration']['Reboosts'].append({
                        "X axis: ":"No reboosts have been detected",
                        "Y axis: ":"No reboosts have been detected",
                        "Z axis: ":"No reboosts have been detected",
                    })
           
            filepath = (path + '/' + str(time.time()) + '.json')
            json_file = json.dumps(json_data, indent = 4, separators=(',', ': '))
            with open(filepath, 'w') as outfile:
                outfile.write(json_file)
           
    except:
        print("Failed to dump data into packet")
    file1 = open("/home/pi/Documents/count.txt", "w")
    internal_count = internal_count + 1
    file1.truncate(0)
    file1.write(str(internal_count))
    time.sleep(2)
   
file1.close()
