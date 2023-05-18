# life-project
import bpy
from math import log
from random import random

gridsize = 30 # количество кубов по x и y
tile_size = 10./gridsize
time_stretch = 3 

class Tile():    
    grid = [[None for x in range(gridsize)] for y in range(gridsize)]
    stepnr = 1
    
    def __init__(self, x, y):
        self.grid[x][y] = self 
        self.x, self.y = x, y
        obj = bpy.data.objects.new("Cube", bpy.data.meshes["Cube"])
        obj.location = (x*tile_size, y*tile_size, 0)
        obj.scale = [tile_size, tile_size, tile_size]
        bpy.data.collections["Grid"].objects.link(obj)
        self.obj = obj
        self.age = 0
        self.nextlife = False
        
        
        # проверка соседних 
    def get_neighbours(self):
        x, y = self.x, self.y 
        l = x-1
        r = x+1 if x < gridsize-1 else 0
        d = y-1
        u = y+1 if y < gridsize-1 else 0
        g = self.grid
        self.neighbors = [g[l][u], g[x][u], g[r][u], g[r][y], 
                          g[r][d], g[x][d], g[l][d], g[l][y]] 
              
        
#        дадим клеткам жизнь/ если условие выполняется, то рандомно увеличивается в 3 раза по z
    def random_life(self):
        if random() < .2: 
            self.nextlife = True
            self.update()
            
    def check(self):       
        sum = len([1 for n in self.neighbors if n.age > 0]) 
        # sum это сумма живых соседей
        if self.age > 0 and (sum < 2 or sum > 3):
             self.nextlife = False
        elif sum == 3:
            self.nextlife = True       
        
        
    def update(self):
        if self.nextlife: # рождение 
            if self.age > 0: # остаётся живой
               self.age += 1
        
            else: # мертва, но потом рождается
                self.age = 1
                self.obj.location.z = log(self.age*1000 + 1) * tile_size/5                   
        
        else: # умирает
            if self.age > 0: # жива, но умрёт
                   self.age = 0
                   
            else: # остаётся мёртвой 
                self.age -= 1
                self.obj.location.z = -log(abs(self.age*1000) + 1) * tile_size/5 
             
             # добавляем keyframe   
            self.obj.keyframe_insert(data_path = "location", index = 2, frame = Tile.stepnr * 4)
 # умножили на 2 и уменьшили скорость, т.к меньше кадров во фрейме
 #    пишем это, чтобы только один объект появлялся       
for obj in bpy.data.collections["Grid"].objects:
    bpy.data.collections["Grid"].objects.unlink(obj)   


#создали сетку 5*5   
tiles = [Tile(x,y) for x in range(gridsize) for y in range(gridsize)]
for t in tiles:
    t.random_life()
    t.get_neighbours()
    
iterations = 300

bpy.context.scene.frame_end = time_stretch*iterations    
# задаём количество фреймов
for i in range(iterations):       
    for t in tiles: t.check()
    for t in tiles: t.update()
    Tile.stepnr += 1
    
