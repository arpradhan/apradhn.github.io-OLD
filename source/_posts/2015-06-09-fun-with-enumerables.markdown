---
layout: post
title: "Fun With Enumerables"
date: 2015-06-09 20:59:28 -0400
comments: true
categories: 
---
Fun with enumerables

Over the last few days I've started experimenting with Enumerables. Nearly every lab required iterating through an array or hash in order to crunch some kind of data, so it made sense to use to utilize Ruby's built-in methods for Enumerble rather than push values to empty array inside an #each loop. 

For example, the first function for holiday.rb of the Apples and Holidays lab requires you to add the supply argument to the each of the winter holidays. To solve this, I looped through the winter hash of the holiday hash, and pushed the new supply to the end of each supply array. Lastly, I entered holiday_hash as the last line of the method so that it would return the updated hash to the program, like so:

holiday_hash[:winter].each do |holiday, supplies|
  supplies << supply
end
holiday_hash

 Since I was returning a hash with identical keys, I realized I could shorten the method by using #collect, like so:

holiday_hash[:winter].collect {|holiday, supplies| supplies << supply }

Last week in the iteration lecture, Avi said something like "Every you want a return a new array that is identical in size to the array you are iterating through, you probably want to use #collect". Since then I've been trying to replace unnecessary #each loops with #collect and in the process found out about some pretty useful enumerables to quickly return data from an array or hash. I learned that #find, #any?, and #select are powerful enumerables for completing some of the most common processes you want to do with an array or hash. So I wanted to spend some time exploring the rest of the Enumerable catalog to see what else could help me get data from an array/hash

#count

Not all enumerables return an array/hash. They can also return boolean values, and in the case of #count, a single number. #count returns a number equal to the amount of times the expression in the block evaluates to true. For example, how many times "raccoon" appears in an array of animals. At least 2 or 3 of the labs required us to get a count of how many times an element recurred in array or hash. This is how you would simulate #count using #each

animals = ["fox", "swan", "raccoon", "turtle", "iguana", "raccoon"]
count = 0
animals.each do |animal|
  if animal == "raccoon"
    count += 1
  end
end
count

#count accomplishes the same logic in a single line:
animals.count{|animal| animal == "raccoon"}

#none?

Sometimes you just want to make sure that a value doesn't exist in your array/hash

animals = {fox: {class: "mammalia"}, lobster: {class: "crustacea"}, eagle: {class: "aves"}, toad: {class: "amphibia"}}
animals.none?{|animal, properties| properties[:class] == "reptilia"}
=> true
 
#none returns true because no animal has a class of repitilia

#one? 

One is a strange enumerator that does exactly what it sounds like it does: returns true if only one expression in the block evaluates to true
animals.one?{|animal, properties| properties[:class] == "aves"}
=> true

#partition is even stranger, because it returns two arrays, one for which the expression in the block evaluates to true, and the other for false. Say I have an arrays of dinosaur hashes, I can use partition to split it into an array of Jurassic and Cretaceous dinosaurs: 


dinosaurs = [
  {name: "Tyrannosaurus", era: "Cretaceous"},
  {name: "Velociraptor", era: "Cretaceous"},
  {name: "Pterodactylus", era: "Jurassic"},
  {name: "Brachiosaurus", era: "Jurassic"},
  {name: "Triceratops", era: "Cretaceous"},
  {name: "Stegasaurus", era: "Jurassic"},
  {name: "Dilophosaurus", era: "Jurassic"}
]

dinosaurs.partition{|dino, properties| properties[:era] == "Jurassic"}
=>  [[{:name=>"Pterodactylus", :era=>"Jurassic"}, {:name=>"Brachiosaurus", :era=>"Jurassic"}, {:name=>"Stegasaurus", :era=>"Jurassic"}, {:name=>"Dilophosaurus", :era=>"Jurassic"}], [{:name=>"Tyrannosaurus", :era=>"Cretaceous"}, {:name=>"Velociraptor", :era=>"Cretaceous"}, {:name=>"Triceratops", :era=>"Cretaceous"}]]

#find_all

#find_all is kind of like #find, except it returns an array containing of all the elements of a collection for which the expression in the block evaluates to true. This is handy in situations where you need to grab all the values from a list that match a specific value. In the following example, I use #find_all to get a list of all the kaiju that first premiered between 1965 and 1975:

kaiju = [
{name: "Godzilla", year: 1954},
{name: "Anguiris", year: 1955},
{name: "Rodan", year: 1956},
{name: "Mothra", year: 1961},
{name: "King Ghidorah", year: 1964},
{name: "Gigan", year: 1972},
{name: "Mothra", year: 1961},
{name: "Mechagodzilla", year: 1974},
{name: "Destoroyah", year: 1995},
{name: "Biollante", year: 1989},
{name: "Gezora", year: 1971}
]

=> [{:name=>"Gigan", :year=>1972}, {:name=>"Mechagodzilla", :year=>1974}, {:name=>"Gezora", :year=>1971}]

#group_by

#group_by is a super powerful enumerable that allows you to reconstruct a collection based on the value in the block. when you use group_by on a collection, it will return a hash with keys named after the value in the block. I imagine that group_by is useful when you have an array or hashes with recurring values. For example, you can reorder an array of hashes by one of its keys: 

games = [
{name: "Super Metroid", console: "Super Nintendo"},
{name: "Toejam and Earl", console: "Sega Genesis"},
{name: "Earthbound", console: "Super Nintendo"},
{name: "Michael Jackson's Moonwalker", console: "Sega Genesis"},
{name: "PaRappa The Rapper", console: "Playstation"},
{name: "Pokémon Snap", console: "Nintendo 64"},
{name: "Tony Hawk's Pro Skater", console: "Playstation"},

games.group_by{|game| game[:console]}

=> {"Super Nintendo"=>[{:name=>"Super Metroid", :console=>"Super Nintendo"}, {:name=>"Earthbound", :console=>"Super Nintendo"}], "Sega Genesis"=>[{:name=>"Toejam and Earl", :console=>"Sega Genesis"}, {:name=>"Michael Jackson's Moonwalker", :console=>"Sega Genesis"}], "Playstation"=>[{:name=>"PaRappa The Rapper", :console=>"Playstation"}, {:name=>"Tony Hawk's Pro Skater", :console=>"Playstation"}], "Nintendo 64"=>[{:name=>"Pokmon Snap", :console=>"Nintendo 64"}, {:name=>"Goldeneye", :console=>"Nintendo 64"}]}
{name: "Goldeneye", console: "Nintendo 64"},
]

In conclusion, enumerables are powerful methods that allow you to extract the most likely data you want to get from an array or hash, if you know how to use them properly. Nearly all of them can be recreated with an #each loop, but enumerables can save numerous lines of code and consolidate the logic of your program. Enumerables seem like a shortcut, but you can end up in a lot of trouble if you don't know exactly what you're using them for. I'm glad we reconstructed some of the enumerables in class last week so that I could get a better understanding of the more exotic enumerables available in Ruby.