import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import numpy as np
import requests
from matplotlib.animation import FuncAnimation

fig = plt.figure()
fig.patch.set_facecolor('black')  # Set the background color to black
ax = fig.add_subplot(111, projection='3d', facecolor='black')
ax.set_axis_off()

# Define the number of latitude and longitude lines
num_lat = 180
num_lon = 360

# Define the radius of the globe
radius = 20000
# Define the radius of the larger globe
larger_radius = 15000  # Adjust this value as needed

# Create a sphere
u = np.linspace(0, 2 * np.pi, num_lon)
v = np.linspace(0, np.pi, num_lat)
x = radius * np.outer(np.cos(u), np.sin(v))
y = radius * np.outer(np.sin(u), np.sin(v))
z = radius * np.outer(np.ones(np.size(u)), np.cos(v))

# Create a larger sphere
u_large = np.linspace(0, 2 * np.pi, num_lon)
v_large = np.linspace(0, np.pi, num_lat)
x_large = larger_radius * np.outer(np.cos(u_large), np.sin(v_large))
y_large = larger_radius * np.outer(np.sin(u_large), np.sin(v_large))
z_large = larger_radius * np.outer(np.ones(np.size(u_large)), np.cos(v_large))

# Plot the sphere
ax.plot_wireframe(x_large, y_large, z_large, color='white', linewidth=0.5)



# Create a scatter plot for the ISS location
iss_scatter = ax.scatter([], [], [], color='red',marker='x',s=50)

# Initialize a list to store the historical positions of the ISS
iss_positions = []

def update(frame):
    # Get the current location of the ISS
    response = requests.get("http://api.open-notify.org/iss-now.json")
    iss_location = response.json()

    # Convert the latitude and longitude to radians
    iss_lat = np.radians(float(iss_location['iss_position']['latitude']))
    iss_lon = np.radians(float(iss_location['iss_position']['longitude']))

    # Convert the latitude and longitude to Cartesian coordinates
    iss_x = radius * np.cos(iss_lat) * np.cos(iss_lon)
    iss_y = radius * np.cos(iss_lat) * np.sin(iss_lon)
    iss_z = radius * np.sin(iss_lat)

    # Append the current position to the list of historical positions
    iss_positions.append((iss_x, iss_y, iss_z))

    # Update the ISS location
    iss_scatter._offsets3d = ([iss_x], [iss_y], [iss_z])

    # Plot the orbit of the ISS
    if len(iss_positions) > 1:
        xs, ys, zs = zip(*iss_positions)
        ax.plot(xs, ys, zs, color='yellow')

    # Make the point fade in and out
    alpha = (np.sin(frame / 0.5) + 1) / 2  # Varies between 0 and 1
    iss_scatter.set_alpha(alpha)

# Create an animation
ani = FuncAnimation(fig, update, frames=np.arange(0,100,0.1), repeat=True)

plt.show()