from controller import Robot

def run_robot():
    # Setup the bot instance and maximum wheel speed
    bot = Robot()
    timestep = int(bot.getBasicTimeStep())
    top_speed = 6.28
    
    # Grab all 8 infrared proximity sensors around the chassis
    sensor_tags = ['ps0', 'ps1', 'ps2', 'ps3', 'ps4', 'ps5', 'ps6', 'ps7']
    sensors = []
    
    for tag in sensor_tags:
        dev = bot.getDevice(tag)
        dev.enable(timestep)
        sensors.append(dev)
        
    # Hook up the wheel motors and set them to continuous velocity mode
    left_wheel = bot.getDevice('left wheel motor')
    right_wheel = bot.getDevice('right wheel motor')
    
    left_wheel.setPosition(float('inf'))
    right_wheel.setPosition(float('inf'))
    left_wheel.setVelocity(0.0)
    right_wheel.setVelocity(0.0)
    
    # Main simulation loop
    while bot.step(timestep) != -1:
        # Pull live readings from every sensor simultaneously
        readings = [s.getValue() for s in sensors]
        
        # Check proximity thresholds for left and right sides
        hit_right = readings[0] > 80.0 or readings[1] > 80.0 or readings[2] > 80.0
        hit_left  = readings[5] > 80.0 or readings[6] > 80.0 or readings[7] > 80.0
        
        # Default cruising speed (half throttle forward)
        v_left = 0.5 * top_speed
        v_right = 0.5 * top_speed
        
        # Reactive steering logic
        if hit_left:
            # Clear left obstacle by pivoting right
            v_left = 0.5 * top_speed
            v_right = -0.5 * top_speed
        elif hit_right:
            # Clear right obstacle by pivoting left
            v_left = -0.5 * top_speed
            v_right = 0.5 * top_speed
            
        # Push the velocities to the motors
        left_wheel.setVelocity(v_left)
        right_wheel.setVelocity(v_right)

if __name__ == '__main__':
    run_robot()
