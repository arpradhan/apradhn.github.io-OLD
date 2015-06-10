---
layout: post
title: "Fun With Enumerables"
date: 2015-06-09 20:59:28 -0400
comments: true
categories: 
---

<p>Over the last few days I've started experimenting with Enumerables. Nearly every lab required iterating through an array or hash in order to crunch some kind of data, so it made sense to use to utilize Ruby's built-in methods for Enumerble rather than push values to empty array inside an <code>#each</code> loop.</p>

<p>For example, the first function for holiday.rb of the Apples and Holidays lab requires you to add the supply argument to the each of the winter holidays. To solve this, I looped through the winter hash of the holiday hash, and pushed the new supply to the end of each supply array. Lastly, I entered holiday_hash as the last line of the method so that it would return the updated hash to the program, like so:</p>

<code>
holiday_hash[:winter].each do |holiday, supplies|
  supplies << supply
end
holiday_hash
</code>

 <p>Since I was returning a hash with identical keys, I realized I could shorten the method by using <code>#collect,</code> like so:</p>

<code>
holiday_hash[:winter].collect {|holiday, supplies| supplies << supply }
</code>

<p>Last week in the iteration lecture, Avi said something like "Every you want a return a new array that is identical in size to the array you are iterating through, you probably want to use #collect". Since then I've been trying to replace unnecessary #each loops with #collect and in the process found out about some pretty useful enumerables to quickly return data from an array or hash. I learned that #find, #any?, and #select are powerful enumerables for completing some of the most common processes you want to do with an array or hash. So I wanted to spend some time exploring the rest of the Enumerable catalog to see what else could help me get data from an array/hash</p>

<h3>#count</h3>

<p>Not all enumerables return an array/hash. They can also return boolean values, and in the case of <code>#count</code>, a single number. <code>#count</code> returns a number equal to the amount of times the expression in the block evaluates to true. For example, how many times "raccoon" appears in an array of animals. At least 2 or 3 of the labs required us to get a count of how many times an element recurred in array or hash. This is how you would simulate #count using <code>#each</code>: </p>

<code>
animals = ["fox", "swan", "raccoon", "turtle", "iguana", "raccoon"]<br>
count = 0<br>
animals.each do |animal|<br>
&nbsp;&nbsp;if animal == "raccoon"<br>
&nbsp;&nbsp;&nbsp;&nbsp;count += 1<br>
&nbsp;&nbsp;end<br>
end<br>
count<br>
</code>

<code>#count</code> accomplishes the same logic in a single line:
<br><code>animals.count{|animal| animal == "raccoon"}</code>

<h3>#none?</h3>

Sometimes you just want to make sure that a value doesn't exist in your array/hash

<code>
animals = {<br>
&nbsp;&nbsp;fox: {class: "mammalia"}, <br>
&nbsp;&nbsp;lobster: {class: "crustacea"}, <br>
&nbsp;&nbsp;eagle: {class: "aves"}, <br>
&nbsp;&nbsp;toad: {class: "amphibia"}}<br>
<br>
animals.none?{|animal, properties| properties[:class] == "reptilia"} <br> #=> true
 </code>

<p><code>#none</code> returns true because no animal has a class of repitilia</p>

<h3>#one?</h3>

<p><code>#one?</code> is a strange enumerator that does exactly what it sounds like it does: returns true if only one expression in the block evaluates to true</p>
<code>
animals.one?{|animal, properties| properties[:class] == "aves"} #=> true
</code>
<h3>#partition</h3>
<p><code>#partition</code> is even stranger, because it returns two arrays, one for which the expression in the block evaluates to true, and the other for false. Say I have an arrays of dinosaur hashes, I can use partition to split it into an array of Jurassic and Cretaceous dinosaurs: </p>

<code>
dinosaurs = [<br>
&nbsp;&nbsp;{name: "Tyrannosaurus", era: "Cretaceous"},<br>
&nbsp;&nbsp;{name: "Velociraptor", era: "Cretaceous"},<br>
&nbsp;&nbsp;{name: "Pterodactylus", era: "Jurassic"},<br>
&nbsp;&nbsp;{name: "Brachiosaurus", era: "Jurassic"},<br>
&nbsp;&nbsp;{name: "Triceratops", era: "Cretaceous"},<br>
&nbsp;&nbsp;{name: "Stegasaurus", era: "Jurassic"},<br>
&nbsp;&nbsp;{name: "Dilophosaurus", era: "Jurassic"}<br>
]<br>
<br>
dinosaurs.partition{|dino, properties| properties[:era] == "Jurassic"}<br> #=>  
[<br>
&nbsp;&nbsp;[{:name=>"Pterodactylus", :era=>"Jurassic"},<br> 
&nbsp;&nbsp;{:name=>"Brachiosaurus", :era=>"Jurassic"},<br> 
&nbsp;&nbsp;{:name=>"Stegasaurus", :era=>"Jurassic"},<br> 
&nbsp;&nbsp;{:name=>"Dilophosaurus", :era=>"Jurassic"}],<br>
<br> 
&nbsp;&nbsp;[{:name=>"Tyrannosaurus", :era=>"Cretaceous"},<br> 
&nbsp;&nbsp;{:name=>"Velociraptor", :era=>"Cretaceous"},<br> 
&nbsp;&nbsp;{:name=>"Triceratops", :era=>"Cretaceous"}]<br>
]</code>

<h3>#find_all</h3>

<p><code>#find_all</code> is kind of like <code>#find</code>, except it returns an array containing of all the elements of a collection for which the expression in the block evaluates to true. This is handy in situations where you need to grab all the values from a list that match a specific value. In the following example, I use <code>#find_all</code> to get a list of all the kaiju that first premiered between 1965 and 1975:</p>
<code>
kaiju = [<br>
&nbsp;&nbsp;{name: "Godzilla", year: 1954},<br>
&nbsp;&nbsp;{name: "Anguiris", year: 1955},<br>
&nbsp;&nbsp;{name: "Rodan", year: 1956},<br>
&nbsp;&nbsp;{name: "Mothra", year: 1961},<br>
&nbsp;&nbsp;{name: "King Ghidorah", year: 1964},<br>
&nbsp;&nbsp;{name: "Gigan", year: 1972},<br>
&nbsp;&nbsp;{name: "Mothra", year: 1961},<br>
&nbsp;&nbsp;{name: "Mechagodzilla", year: 1974},<br>
&nbsp;&nbsp;{name: "Destoroyah", year: 1995},<br>
&nbsp;&nbsp;{name: "Biollante", year: 1989},<br>
&nbsp;&nbsp;{name: "Gezora", year: 1971}<br>
]<br> 
kaiju.find_all {|k| k[:year] > 1965 && k[:year] < 1975 } #=> [<br>
&nbsp;&nbsp;{:name=>"Gigan", :year=>1972}, <br>
&nbsp;&nbsp;{:name=>"Mechagodzilla", :year=>1974}, <br>
&nbsp;&nbsp;{:name=>"Gezora", :year=>1971}<br>]
</code>

<h3>#group_by</h3>

<p><code>#group_by</code> is a super powerful enumerable that allows you to reconstruct a collection based on the value in the block. when you use group_by on a collection, it will return a hash with keys named after the value in the block. I imagine that group_by is useful when you have an array or hashes with recurring values. For example, you can reorder an array of hashes by one of its keys: </p>
<code>
games = [<br>
{name: "Super Metroid", console: "Super Nintendo"},<br>
{name: "Toejam and Earl", console: "Sega Genesis"},<br>
{name: "Earthbound", console: "Super Nintendo"},<br>
{name: "Michael Jackson's Moonwalker", console: "Sega Genesis"},<br>
{name: "PaRappa The Rapper", console: "Playstation"},<br>
{name: "Pokémon Snap", console: "Nintendo 64"},<br>
{name: "Tony Hawk's Pro Skater", console: "Playstation"}<br>
{name: "Goldeneye", console: "Nintendo 64"}<br>
]
<br>
<br>
games.group_by{|game| game[:console]}
</code>

returns:
<br>
<code>
 {"Super Nintendo"=>[<br>
 &nbsp;&nbsp;{:name=>"Super Metroid", :console=>"Super Nintendo"},<br>
 &nbsp;&nbsp;{:name=>"Earthbound", :console=>"Super Nintendo"}<br>
 ], <br>
 "Sega Genesis"=>[<br>
 &nbsp;&nbsp;{:name=>"Toejam and Earl", :console=>"Sega Genesis"},<br>
 &nbsp;&nbsp;{:name=>"Michael Jackson's Moonwalker", :console=>"Sega Genesis"}<br>
 ],<br>
"Playstation"=>[<br>
&nbsp;&nbsp;{:name=>"PaRappa The Rapper", :console=>"Playstation"},<br>
&nbsp;&nbsp;{:name=>"Tony Hawk's Pro Skater", :console=>"Playstation"}],<br>
"Nintendo 64"=>[<br>
&nbsp;&nbsp;{:name=>"Pokmon Snap", :console=>"Nintendo 64"},<br>
&nbsp;&nbsp;{:name=>"Goldeneye", :console=>"Nintendo 64"}<br>
]}
</code>

In conclusion, enumerables are powerful methods that allow you to extract the most likely data you want to get from an array or hash, if you know how to use them properly. Nearly all of them can be recreated with an #each loop, but enumerables can save numerous lines of code and consolidate the logic of your program. Enumerables seem like a shortcut, but you can end up in a lot of trouble if you don't know exactly what you're using them for. I'm glad we reconstructed some of the enumerables in class last week so that I could get a better understanding of the more exotic enumerables available in Ruby.