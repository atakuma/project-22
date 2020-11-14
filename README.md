# project-22
# load the project support code
include shared-gdrive(
  "cs111-2020.arr",
  "1imMXJxpNWFCUaawtzIJzPhbDuaLHtuDX")

include shared-gdrive(
  "project-2-support.arr",
  "1N1pwFonshMA_CH99wH00h0HuuiXRja9A")

include image
include tables
include reactors
import lists as L

ssid = "1Jt90nPp_qpoFg5oRFwiBCSWJWCZbecWsRHnWxbtwYGY"
maze-data = load-maze(ssid)
item-data = load-items(ssid)

data Posn:
  | posn(x :: Number, y :: Number)
end

data Player:
  | new-player(img :: Image, pos :: Posn, portals :: Number)
end

data Portal:
  | new-portal(img :: Image, pos :: Posn)
end

data GameState:
  | game(player :: Player, portal-table :: Table, portals-on-screen :: Image)
end

init-posn = posn(45, 45)

doug = new-player(load-texture("doug-down.png"), init-posn, 0)

item-data-length-range = range(0, item-data.length()) 

wall = load-texture("walls.png")
tile = load-texture("tile.png")
portal = load-texture("pop-rocks.png")


fun create-row(ro :: List<String>) -> Image:
  doc: "inputs list of strings to create horizontal pieces of maze"
  cases (List) ro:
    | empty => empty-image
    | link(f, r) => 
      if (f == "x"):
        beside(wall, create-row(r))
      else:
        beside(tile, create-row(r))
      end
  end
where: 
  create-row([list: "x", "o", "x"]) is beside((beside(wall, tile)), wall)
  create-row([list: "x", "a", "x"]) is beside((beside(wall, tile)), wall)
  create-row([list: ]) is empty-image
end

test-list1 = [list: [list: "o", "x", "o"], [list: "x", "o", "x"]]

fun create-maze(maze :: List<List<String>>) -> Image:
  doc: "inputs list of lists of strings to piece together horizontal images and output maze image"
  cases (List) maze:
    | empty => empty-image
    | link(f, r) =>
      above(create-row(f), create-maze(r))
  end
where:
  create-maze(test-list1) is above((beside((beside(tile, wall)), tile)), 
    beside((beside(wall, tile)), wall))
  create-maze([list: ]) is empty-image
end

maze-fin = create-maze(maze-data)

fun draw-portals-2(table-length :: List<Number>) -> Image:
  doc: "draws portals on map given spreadsheet"
  cases (List) table-length:
    | empty => maze-fin
    | link(f, r) =>
      place-image(load-texture(get-row(item-data, f)["url"]), 
        (get-row(item-data, f)["x"] * 30) + 15,
        (get-row(item-data, f)["y"] * 30) + 15, draw-portals-2(r))
  end
end
  

init-state = game(doug, item-data, draw-portals-2(item-data-length-range))


fun draw-portals(lop :: List<Portal>) -> Image:
  doc: "places portals on map"
  cases (List) lop:
    | empty => maze-fin
    | link(f, r) =>
      place-image(f.img, f.pos.x, f.pos.y, draw-portals(r))
  end
end

fun draw-game(state :: GameState) -> Image:
  #when does draw-game run
  place-image(state.player.img, state.player.pos.x, state.player.pos.y, state.portals-on-screen)
    
    #draw-portals-2(item-data-length-range))
  #state.portals-on-screen
end



#draw-game(init-state)

fun evaluate-position(state-3 :: GameState) -> GameState:
  pport = state-3.player.portals
  fun draw-portals-3(table-length :: List<Number>) -> Image:
    doc: "this redraws the portals after someone lands on them"
        cases (List) table-length:
          | empty => maze-fin
          | link(f, r) =>
            place-image(load-texture(get-row(filter-with(state-3.portal-table,
                check-if-on-portal-2), f)["url"]), (get-row(filter-with(state-3.portal-table, 
                check-if-on-portal-2), f)["x"] * 30) + 15, 
          (get-row(filter-with(state-3.portal-table, 
                check-if-on-portal-2), f)["y"] * 30) + 15, draw-portals-3(r))
        end
      end
      
      fun check-if-on-portal-2(row :: Row) -> Boolean:
    doc: "checks if player is on the portal and deletes portal if true"
          not((row["x"] == get-maze-index(state-3.player.pos.x)) and (row["y"] == 
        get-maze-index(state-3.player.pos.y)))
        end
       if (filter-with(state-3.portal-table, check-if-on-portal-2).length() == 
      state-3.portal-table.length()): 
        #if player is not on a portal
      state-3
       else: 
        #if player is on a portal
        #this is going to delete the portal from the game
        game(new-player(state-3.player.img, posn(state-3.player.pos.x, state-3.player.pos.y),
        pport + 1), filter-with(state-3.portal-table, check-if-on-portal-2), 
      draw-portals-3(range(0, filter-with(state-3.portal-table, check-if-on-portal-2).length())))
        end
end

example-table = table: name :: String, x :: Number, y :: Number, url :: String
  row: "Pop Rocks", 8, 8, "pop-rocks.png"
end

fun key-pressed(state :: GameState, key :: String) -> GameState:
  doc: "changes player position corresponding to wasd keys"
  posx = state.player.pos.x
  tilex = (state.player.pos.x - 15) / 30
  posy = state.player.pos.y
  tiley = (state.player.pos.y - 15) / 30
  pimg = state.player.img
  pport = state.player.portals
  #table-length-range = range(0, item-data.length())

  if (key == "w") and (maze-data.get(tiley - 1).get(tilex) == "o"):
      evaluate-position(game(new-player(pimg, posn(posx, posy - 30), pport), state.portal-table, 
        state.portals-on-screen))
  else if (key == "a") and (maze-data.get(tiley).get(tilex - 1) == "o"):
      evaluate-position(game(new-player(pimg, posn(posx - 30, posy), pport), state.portal-table, 
        state.portals-on-screen))
  else if (key == "s") and (maze-data.get(tiley + 1).get(tilex) == "o"):
      evaluate-position(game(new-player(pimg, posn(posx, posy + 30), pport), state.portal-table, 
        state.portals-on-screen))
  else if (key == "d") and (maze-data.get(tiley).get(tilex + 1) == "o"):
      evaluate-position(game(new-player(pimg, posn(posx + 30, posy), pport), state.portal-table, 
        state.portals-on-screen))
  else:
    state
  end
where:
  key-pressed(game(new-player(load-texture("doug-down.png"), posn((8 * 30) + 15, (8 * 30) + 15), 0), 
        example-table, portal), "w") is game(new-player(load-texture("doug-down.png"), 
      posn((8 * 30) + 15, (8 * 30) + 15), 0), example-table, portal)
  key-pressed(game(new-player(load-texture("doug-down.png"), posn((5 * 30) - 15, (8 * 30) + 15), 0), 
        example-table, portal), "w") is game(new-player(load-texture("doug-down.png"), 
      posn((5 * 30) - 15, (8 * 30) - 15), 0), example-table, portal)
  key-pressed(game(new-player(load-texture("doug-down.png"), posn((5 * 30) - 15, (8 * 30) + 15), 0), 
      example-table, portal), "r") is game(new-player(load-texture("doug-down.png"), 
      posn((5 * 30) - 15, (8 * 30) + 15), 0), example-table, portal)
  key-pressed(game(new-player(load-texture("doug-down.png"), posn((5 * 30) - 15, (8 * 30) + 15), 0), 
      example-table, portal), "s") is game(new-player(load-texture("doug-down.png"), 
      posn((5 * 30) - 15, (8 * 30) + 45), 0), example-table, portal)
  key-pressed(game(new-player(load-texture("doug-down.png"), posn((5 * 30) - 15, (8 * 30) + 15), 0), 
      example-table, portal), "d") is game(new-player(load-texture("doug-down.png"), 
      posn((5 * 30) + 15, (8 * 30) + 15), 0), example-table, portal)
  key-pressed(game(new-player(load-texture("doug-down.png"), posn((5 * 30) - 15, (8 * 30) + 15), 0), 
      example-table, portal), "a") is game(new-player(load-texture("doug-down.png"), 
      posn((5 * 30) - 45, (8 * 30) + 15), 0), example-table, portal)
end
#***TOMORROW***
#Tomorrow, you have to make the portal disapear when you land on it
#Do I redraw the portals after everytime I hit one? How? 
#***I THINK YOU SHOULD JUST PLACE A TILE IMAGE OVER IT! MIGHT BE THAT EASY***
#I think you should think about using the portal data-type and 
#Also, make sure you can only click if you have a portal. And subtract a portal on every successful click. But that will be easy, the disapeaing part will be the hard one. 
#I think you have to use a data-type that includes portals
#add portal images to game-state. And redraw using draw-portals-2 after every time you land on a portal. This will not include the portal you landed on cause it won't be in the table



fun use-portal(state :: GameState, posx-click :: Number, posy-click :: Number, 
    mouse-event :: String) -> GameState:
  doc: "inputs mouse event and location and updates gamestate to move character to new location"
  tilex = (state.player.pos.x - 15) / 30
  tiley = (state.player.pos.y - 15) / 30
  pimg = state.player.img
  pport = state.player.portals
  posx = state.player.pos.x
  posy = state.player.pos.y
  distance = num-sqrt(num-sqr(get-maze-index(posx-click) - get-maze-index(posx)) + 
    num-sqr(get-maze-index(posy-click) - get-maze-index(posy)))
  portal-table = state.portal-table
  
  if (mouse-event == "button-down") and 
    (maze-data.get(get-maze-index(posy-click)).get(get-maze-index(posx-click)) == "o") and 
    (distance < 3.5) and (state.player.portals >= 1):
    evaluate-position(game(new-player(pimg, posn((get-maze-index(posx-click) * 30) + 15, 
            (get-maze-index(posy-click) * 30) + 15), pport - 1), portal-table, state.portals-on-screen))
  else:
    state
  end
  where: 
  use-portal(game(new-player(load-texture("doug-down.png"), posn(47, 48), 1), example-table, portal),
    48, 48, "button-down") is 
  game(new-player(load-texture("doug-down.png"), posn(48, 48), 0), example-table, portal)
end
#If portal collected -> add 1 to num-portals and take that portal off the table. Then redraw portals. 

fun game-complete(state :: GameState) -> Boolean:
  doc: "true when player reaches end of maze"
  (get-maze-index(state.player.pos.x) == (get-row(item-data, item-data.length() - 1)["x"])) and 
  (get-maze-index(state.player.pos.y) == get-row(item-data, item-data.length() - 1)["y"])
end

maze-game =
  reactor:
    init              : init-state,
    to-draw           : draw-game,
    on-key            : key-pressed,
    on-mouse          : use-portal,
    stop-when         : game-complete, # [up to you]
    close-when-stop   : true, # [up to you]
    title             : "Captured by Candy!" # [you can change this title]
  end

interact(maze-game)

#| 
   1. Representing the maze layout as a list-of-lists of strings rather than a table made it easy
   to use recrusive functions in order to make the background of the image by recursively building 
   the image. The list representation of the maze layout made it harder to visualize than a table
   representation would have, but a table representation would have been more difficult to
   manipulate in order to produce images.
   2. The Google Sheets format important for visualizing what the maze image should look like before 
   it was created. The Google Sheets format was not helpful for actually applying code in order to 
   produce the image and run functions on. The list-of-lists version was the most difficult to 
   visualize but the best for actually creating the gmae image and using for functions. The image 
   itself was good for visually testing the game reactor. Running the game and seeing how functions
   interacted if at all was one of the most valuable ways to test the game. 
   3. Insights gained:
   Partner 1: I gained an insight on how a real game might store values (number of portals) as well as
   how new and different data types can be created to make storing and updating values easier to 
   code.
   Partner 2: In this project, I learned that using datatypes can be extremely helpful. In the past,
   there have been numerous times in which I would have liked to have a function output two different 
   types of data, such as an image and a position, yet was unable to. However, in this project, 
   I learned that datatypes can allow a function to update different types of data in a datatype. 
   I will definitely use datatypes more often in the future.
   Partner3 : I learned that custom data types are important and that you have to utilize them in 
   order to use reactor.
   4. Misconceptions/mistakes worked throug:
   The two major misconceptions we worked through were that we did not quite understand how to
   manipulate portals to disappear when the player walked over them and how to store the number of
   portals the player is carrying. 
   5. Followup questions:
   Our questions are regarding the uses and complexity of data types. Data types seem like
   a very useful tool for simplifying later functions, but they are themselves fairly complicated. 
   What are all the ways in which data types are usable? How are data types used in the programming
   world? How complicated can data types become?
|#
