# 助记词种子
将随机数字编码成单词，并使用它们创建种子。
[BIP 39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

助记符句（“助记码”，“种子短语”，“种子词语”）是一种将大量随机生成的数字表示为一系列单词的方法，使人们更容易存储。

然后使用这些单词创建种子，可用于在[分层确定性钱包](../HD%20Wallets.md)中生成[扩展密钥](../Extended%20Keys/Extended%20Keys.md)。

>**注意**：换句话说，你的[钱包](../../../Beginners/Getting%20Started/getting-started/getting%20started.md)需要使用一个大的随机数来生成所有的密钥和地址，而助记符句为我们提供了一种方便的存储方式。

>**不要相信在网站上生成的种子**。创建钱包时，应私下在自己的计算机上生成自己的种子。我建议使用 [Electrum](https://electrum.org/)。

## 如何生成助记句？

三个步骤：

1. 生成熵
2. 将熵转换为助记句
3. 将助记句转换为种子

### 1.生成熵
![Mnemonic Seed-1.png](img/Mnemonic%20Seed-1%20(1).gif)
你的熵应为128到256位。

要创建助记词，首先需要生成熵，这是我们的随机源。

你可以将熵视为一个非常大的随机数，没有人以前或以后会生成它。最好将此数字视为一系列位（例如10011010010001…），这是计算机存储数字的方式。

>**提示**：位（0或1）是计算机上最小的存储单位。

我们生成的熵必须是32位的倍数，因为这将允许我们将熵分成均匀的块并在以后将其转换为单词。此外，熵应该**介于128到256位之间**，因为这足以使两个人无法生成相同的熵。
```ruby
# -------------------
# 1. Generate Entropy
# -------------------
require 'securerandom' # library for generating bytes of entropy

bytes = SecureRandom.random_bytes(16) # 16 bytes = 128 bits (1 byte = 8 bits)
entropy = bytes.unpack("B*").join     # convert bytes to a string of bits (base2)
puts entropy #=> "1010110111011000110010010010111001001011001001010110001011100001"

# Note: For the purposes of the examples on this page, I have actually generated 64 bits of entropy.
# For real world use, you should generate 128 to 256 bits (in a multiple of 32 bits).
```
>**注意**：始终使用安全的随机数生成器来生成熵。不要使用编程语言的默认“随机”函数，因为它产生的数字对于密码学来说不够随机。

### 2. 熵到助记词
现在我们已经有了我们的熵，我们可以将其编码成单词。

首先，我们为我们的熵添加一个校验和，以帮助检测错误（使最终句子对用户更友好）。这个校验和是通过将熵哈希通过SHA256创建的，这为我们的熵提供了一个唯一的指纹。然后，我们取该哈希的1位，用于每32位熵，并将其添加到我们的熵的末尾。
![Mnemonic Seed-2.png](img/Mnemonic%20Seed-2%20(1).png)

接下来，我们将它分成11位一组，将这些数字转换为十进制数，并使用这些数字来选择相应的单词。
![Mnemonic Seed-3.png](img/Mnemonic%20Seed-3%20(1).png)
这个单词列表中有2048个单词。
这就是我们的助记句。

提示：11位数字可以存储0-2047之间的小数（这就是为什么单词列表中有2048个单词）。

提示：通过在每32位熵中添加1位校验和，我们将始终得到33位的倍数，我们可以将其分成相等的11位块。

注意：助记短语通常为**12到24个单词**。
```
# ----------------------
# 2. Entropy to Mnemonic
# ----------------------
entropy = "1010110111011000110010010010111001001011001001010110001011100001"

# 1. Create checksum
require 'digest'
size = entropy.length / 32 # number of bits to take from hash of entropy (1 bit checksum for every 32 bits entropy)
sha256 = Digest::SHA256.digest([entropy].pack("B*")) # hash of entropy (in raw binary)
checksum = sha256.unpack("B*").join[0..size-1] # get desired number of bits
puts "checksum: #{checksum}"

# 2. Combine
full = entropy + checksum
puts "combined: #{full}"

# 3. Split in to strings of of 11 bits
pieces = full.scan(/.{11}/)

# 4. Get the wordlist as an array
wordlist = File.readlines("wordlist.txt")

# 5. Convert groups of bits to array of words
puts "words:"
sentence = []
pieces.each do |piece|
  i = piece.to_i(2)   # convert string of 11 bits to an integer
  word = wordlist[i]  # get the corresponding word from wordlist
  sentence << word.chomp # add to sentence (removing newline from end of word)
  puts "#{piece} #{i.to_s.rjust(4)} #{word}"
end

mnemonic = sentence.join(" ")
puts "mnemonic: #{mnemonic}" #=> "punch shock entire north file identify"
```

### 3. 从助记词到种子
现在我们有了助记词，我们可以将其转换成最终的种子。

要创建种子，需要将助记词通过 PBKDF2 函数。这基本上是将您的助记词（+ 可选的口令）多次哈希，直到产生最终的 64 字节（512 位）结果。
![Mnemonic Seed-4.png](img/Mnemonic%20Seed-4%20(1).gif)
可选的口令短语允许你修改最终种子。

这个64字节的结果就是你的种子，可以用来创建分层确定性钱包的[主扩展密钥](../Extended%20Keys/Extended%20Keys.md)。
```ruby
# -------------------
# 3. Mnemonic to Seed
# -------------------
require 'openssl'

mnemonic = "punch shock entire north file identify"
passphrase = "" # can leave this blank
puts "passphrase: #{passphrase}"

password = mnemonic
salt = "mnemonic#{passphrase}" # "mnemonic" is always used in the salt with optional passphrase appended to it
iterations = 2048
keylength = 64
digest = OpenSSL::Digest::SHA512.new

result = OpenSSL::PKCS5.pbkdf2_hmac(password, salt, iterations, keylength, digest)
seed = result.unpack("H*")[0] # convert to hexadecimal string
puts "seed: #{seed}" #=> e1ca8d8539fb054eda16c35dcff74c5f88202b88cb03f2824193f4e6c5e87dd2e24a0edb218901c3e71e900d95e9573d9ffbf870b242e927682e381d109ae882
```

>PBKDF2 Settings:
Password: Mnemonic Sentence
Salt: "mnemonic"+(optional passphrase)
Iterations: 2048
Algorithm: HMAC-SHA512
Size: 64 bytes

>[PBKDF2](https://en.wikipedia.org/wiki/PBKDF2) - **基于密码的密钥派生函数2**
PBKDF2基本上是一个哈希函数，旨在通过多次哈希数据来减慢计算速度，从而使得对助记语句进行暴力破解以获取人们实际使用的种子更加困难。
此外，PBKDF2还允许你提供第二个输入，称为盐（“口令短语”，“种子扩展”），以及你要哈希的数据，这使能够从相同的助记语句产生完全不同的种子。
>>链接。
* [PBKDF和SHA512之间有什么区别？](https://crypto.stackexchange.com/questions/35275/whats-the-difference-between-pbkdf-and-sha-and-why-use-them-together)
* [使用PBKDF2与SHA256相比有什么优势？](https://security.stackexchange.com/questions/16354/whats-the-advantage-of-using-pbkdf2-vs-sha256-to-generate-an-aes-encryption-key)

**验证助记词**

助记词句子包含一个校验和，这意味着可以**检查给定的助记词句子是否有效**。

为此，你将助记词中的单词转换回位，确定熵部分和校验和部分，并检查从熵中创建的校验和是否与助记词中的校验和匹配。
![Mnemonic Seed-5.png](img/Mnemonic%20Seed-5%20(1).png)

>**警告**：不要随机生成12-24个单词来创建助记词句子。如果你只是随机选择12-24个单词，那么你最后选择的单词可能不包含正确的校验和，导致当你尝试导入到钱包时会显示无效。
```ruby
# -----------------
# Validate Mnemonic
# -----------------
require 'digest'

mnemonic = "punch shock entire north file identify"
puts "mnemonic: #{mnemonic}"

# Get the wordlist
wordlist = File.readlines("wordlist.txt").map { |word| word.chomp} # remove new lines from end of each word

# Convert mnemonic to binary string
binary = ""
mnemonic.split(" ").each do |word|
    i = wordlist.index(word)       # get word index number in wordlist
    bin = i.to_s(2).rjust(11, "0") # convert index number to an 11-bit number
    binary << bin                  # add to binary string
end
puts "entropy+checksum: #{binary}"

# Get the original entropy and checksum
x = binary.length / 33 # for every 33 bits, 1 bit will be checksum
entropy  = binary[0...-x] # first part is original entropy
checksum = binary[-x..-1] # last x bits are checksum

# Compute the expected checksum from the entropy
hash = Digest::SHA256.digest([entropy].pack("B*"))
checksumexpected = hash.unpack("B*").join[0...x]

# Check the checksum in the mnemonic sentence matches the checksum of the entropy
puts "checksum: #{checksum}"
puts "expected: #{checksumexpected}"
puts "valid: #{checksum == checksumexpected}" #=> true
```
Requires wordlist.txt

BIP39单词列表
一个助记句中的单词来自一个固定的2048个单词列表（由[BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039/bip-0039-wordlists.md)指定）。每个单词的前4个字母在列表中是唯一的。
```
abandon
ability
able
about
above
absent
absorb
abstract
absurd
abuse
access
accident
account
accuse
achieve
acid
acoustic
acquire
across
act
action
actor
actress
actual
adapt
add
addict
address
adjust
admit
adult
advance
advice
aerobic
affair
afford
afraid
again
age
agent
agree
ahead
aim
air
airport
aisle
alarm
album
alcohol
alert
alien
all
alley
allow
almost
alone
alpha
already
also
alter
always
amateur
amazing
among
amount
amused
analyst
anchor
ancient
anger
angle
angry
animal
ankle
announce
annual
another
answer
antenna
antique
anxiety
any
apart
apology
appear
apple
approve
april
arch
arctic
area
arena
argue
arm
armed
armor
army
around
arrange
arrest
arrive
arrow
art
artefact
artist
artwork
ask
aspect
assault
asset
assist
assume
asthma
athlete
atom
attack
attend
attitude
attract
auction
audit
august
aunt
author
auto
autumn
average
avocado
avoid
awake
aware
away
awesome
awful
awkward
axis
baby
bachelor
bacon
badge
bag
balance
balcony
ball
bamboo
banana
banner
bar
barely
bargain
barrel
base
basic
basket
battle
beach
bean
beauty
because
become
beef
before
begin
behave
behind
believe
below
belt
bench
benefit
best
betray
better
between
beyond
bicycle
bid
bike
bind
biology
bird
birth
bitter
black
blade
blame
blanket
blast
bleak
bless
blind
blood
blossom
blouse
blue
blur
blush
board
boat
body
boil
bomb
bone
bonus
book
boost
border
boring
borrow
boss
bottom
bounce
box
boy
bracket
brain
brand
brass
brave
bread
breeze
brick
bridge
brief
bright
bring
brisk
broccoli
broken
bronze
broom
brother
brown
brush
bubble
buddy
budget
buffalo
build
bulb
bulk
bullet
bundle
bunker
burden
burger
burst
bus
business
busy
butter
buyer
buzz
cabbage
cabin
cable
cactus
cage
cake
call
calm
camera
camp
can
canal
cancel
candy
cannon
canoe
canvas
canyon
capable
capital
captain
car
carbon
card
cargo
carpet
carry
cart
case
cash
casino
castle
casual
cat
catalog
catch
category
cattle
caught
cause
caution
cave
ceiling
celery
cement
census
century
cereal
certain
chair
chalk
champion
change
chaos
chapter
charge
chase
chat
cheap
check
cheese
chef
cherry
chest
chicken
chief
child
chimney
choice
choose
chronic
chuckle
chunk
churn
cigar
cinnamon
circle
citizen
city
civil
claim
clap
clarify
claw
clay
clean
clerk
clever
click
client
cliff
climb
clinic
clip
clock
clog
close
cloth
cloud
clown
club
clump
cluster
clutch
coach
coast
coconut
code
coffee
coil
coin
collect
color
column
combine
come
comfort
comic
common
company
concert
conduct
confirm
congress
connect
consider
control
convince
cook
cool
copper
copy
coral
core
corn
correct
cost
cotton
couch
country
couple
course
cousin
cover
coyote
crack
cradle
craft
cram
crane
crash
crater
crawl
crazy
cream
credit
creek
crew
cricket
crime
crisp
critic
crop
cross
crouch
crowd
crucial
cruel
cruise
crumble
crunch
crush
cry
crystal
cube
culture
cup
cupboard
curious
current
curtain
curve
cushion
custom
cute
cycle
dad
damage
damp
dance
danger
daring
dash
daughter
dawn
day
deal
debate
debris
decade
december
decide
decline
decorate
decrease
deer
defense
define
defy
degree
delay
deliver
demand
demise
denial
dentist
deny
depart
depend
deposit
depth
deputy
derive
describe
desert
design
desk
despair
destroy
detail
detect
develop
device
devote
diagram
dial
diamond
diary
dice
diesel
diet
differ
digital
dignity
dilemma
dinner
dinosaur
direct
dirt
disagree
discover
disease
dish
dismiss
disorder
display
distance
divert
divide
divorce
dizzy
doctor
document
dog
doll
dolphin
domain
donate
donkey
donor
door
dose
double
dove
draft
dragon
drama
drastic
draw
dream
dress
drift
drill
drink
drip
drive
drop
drum
dry
duck
dumb
dune
during
dust
dutch
duty
dwarf
dynamic
eager
eagle
early
earn
earth
easily
east
easy
echo
ecology
economy
edge
edit
educate
effort
egg
eight
either
elbow
elder
electric
elegant
element
elephant
elevator
elite
else
embark
embody
embrace
emerge
emotion
employ
empower
empty
enable
enact
end
endless
endorse
enemy
energy
enforce
engage
engine
enhance
enjoy
enlist
enough
enrich
enroll
ensure
enter
entire
entry
envelope
episode
equal
equip
era
erase
erode
erosion
error
erupt
escape
essay
essence
estate
eternal
ethics
evidence
evil
evoke
evolve
exact
example
excess
exchange
excite
exclude
excuse
execute
exercise
exhaust
exhibit
exile
exist
exit
exotic
expand
expect
expire
explain
expose
express
extend
extra
eye
eyebrow
fabric
face
faculty
fade
faint
faith
fall
false
fame
family
famous
fan
fancy
fantasy
farm
fashion
fat
fatal
father
fatigue
fault
favorite
feature
february
federal
fee
feed
feel
female
fence
festival
fetch
fever
few
fiber
fiction
field
figure
file
film
filter
final
find
fine
finger
finish
fire
firm
first
fiscal
fish
fit
fitness
fix
flag
flame
flash
flat
flavor
flee
flight
flip
float
flock
floor
flower
fluid
flush
fly
foam
focus
fog
foil
fold
follow
food
foot
force
forest
forget
fork
fortune
forum
forward
fossil
foster
found
fox
fragile
frame
frequent
fresh
friend
fringe
frog
front
frost
frown
frozen
fruit
fuel
fun
funny
furnace
fury
future
gadget
gain
galaxy
gallery
game
gap
garage
garbage
garden
garlic
garment
gas
gasp
gate
gather
gauge
gaze
general
genius
genre
gentle
genuine
gesture
ghost
giant
gift
giggle
ginger
giraffe
girl
give
glad
glance
glare
glass
glide
glimpse
globe
gloom
glory
glove
glow
glue
goat
goddess
gold
good
goose
gorilla
gospel
gossip
govern
gown
grab
grace
grain
grant
grape
grass
gravity
great
green
grid
grief
grit
grocery
group
grow
grunt
guard
guess
guide
guilt
guitar
gun
gym
habit
hair
half
hammer
hamster
hand
happy
harbor
hard
harsh
harvest
hat
have
hawk
hazard
head
health
heart
heavy
hedgehog
height
hello
helmet
help
hen
hero
hidden
high
hill
hint
hip
hire
history
hobby
hockey
hold
hole
holiday
hollow
home
honey
hood
hope
horn
horror
horse
hospital
host
hotel
hour
hover
hub
huge
human
humble
humor
hundred
hungry
hunt
hurdle
hurry
hurt
husband
hybrid
ice
icon
idea
identify
idle
ignore
ill
illegal
illness
image
imitate
immense
immune
impact
impose
improve
impulse
inch
include
income
increase
index
indicate
indoor
industry
infant
inflict
inform
inhale
inherit
initial
inject
injury
inmate
inner
innocent
input
inquiry
insane
insect
inside
inspire
install
intact
interest
into
invest
invite
involve
iron
island
isolate
issue
item
ivory
jacket
jaguar
jar
jazz
jealous
jeans
jelly
jewel
job
join
joke
journey
joy
judge
juice
jump
jungle
junior
junk
just
kangaroo
keen
keep
ketchup
key
kick
kid
kidney
kind
kingdom
kiss
kit
kitchen
kite
kitten
kiwi
knee
knife
knock
know
lab
label
labor
ladder
lady
lake
lamp
language
laptop
large
later
latin
laugh
laundry
lava
law
lawn
lawsuit
layer
lazy
leader
leaf
learn
leave
lecture
left
leg
legal
legend
leisure
lemon
lend
length
lens
leopard
lesson
letter
level
liar
liberty
library
license
life
lift
light
like
limb
limit
link
lion
liquid
list
little
live
lizard
load
loan
lobster
local
lock
logic
lonely
long
loop
lottery
loud
lounge
love
loyal
lucky
luggage
lumber
lunar
lunch
luxury
lyrics
machine
mad
magic
magnet
maid
mail
main
major
make
mammal
man
manage
mandate
mango
mansion
manual
maple
marble
march
margin
marine
market
marriage
mask
mass
master
match
material
math
matrix
matter
maximum
maze
meadow
mean
measure
meat
mechanic
medal
media
melody
melt
member
memory
mention
menu
mercy
merge
merit
merry
mesh
message
metal
method
middle
midnight
milk
million
mimic
mind
minimum
minor
minute
miracle
mirror
misery
miss
mistake
mix
mixed
mixture
mobile
model
modify
mom
moment
monitor
monkey
monster
month
moon
moral
more
morning
mosquito
mother
motion
motor
mountain
mouse
move
movie
much
muffin
mule
multiply
muscle
museum
mushroom
music
must
mutual
myself
mystery
myth
naive
name
napkin
narrow
nasty
nation
nature
near
neck
need
negative
neglect
neither
nephew
nerve
nest
net
network
neutral
never
news
next
nice
night
noble
noise
nominee
noodle
normal
north
nose
notable
note
nothing
notice
novel
now
nuclear
number
nurse
nut
oak
obey
object
oblige
obscure
observe
obtain
obvious
occur
ocean
october
odor
off
offer
office
often
oil
okay
old
olive
olympic
omit
once
one
onion
online
only
open
opera
opinion
oppose
option
orange
orbit
orchard
order
ordinary
organ
orient
original
orphan
ostrich
other
outdoor
outer
output
outside
oval
oven
over
own
owner
oxygen
oyster
ozone
pact
paddle
page
pair
palace
palm
panda
panel
panic
panther
paper
parade
parent
park
parrot
party
pass
patch
path
patient
patrol
pattern
pause
pave
payment
peace
peanut
pear
peasant
pelican
pen
penalty
pencil
people
pepper
perfect
permit
person
pet
phone
photo
phrase
physical
piano
picnic
picture
piece
pig
pigeon
pill
pilot
pink
pioneer
pipe
pistol
pitch
pizza
place
planet
plastic
plate
play
please
pledge
pluck
plug
plunge
poem
poet
point
polar
pole
police
pond
pony
pool
popular
portion
position
possible
post
potato
pottery
poverty
powder
power
practice
praise
predict
prefer
prepare
present
pretty
prevent
price
pride
primary
print
priority
prison
private
prize
problem
process
produce
profit
program
project
promote
proof
property
prosper
protect
proud
provide
public
pudding
pull
pulp
pulse
pumpkin
punch
pupil
puppy
purchase
purity
purpose
purse
push
put
puzzle
pyramid
quality
quantum
quarter
question
quick
quit
quiz
quote
rabbit
raccoon
race
rack
radar
radio
rail
rain
raise
rally
ramp
ranch
random
range
rapid
rare
rate
rather
raven
raw
razor
ready
real
reason
rebel
rebuild
recall
receive
recipe
record
recycle
reduce
reflect
reform
refuse
region
regret
regular
reject
relax
release
relief
rely
remain
remember
remind
remove
render
renew
rent
reopen
repair
repeat
replace
report
require
rescue
resemble
resist
resource
response
result
retire
retreat
return
reunion
reveal
review
reward
rhythm
rib
ribbon
rice
rich
ride
ridge
rifle
right
rigid
ring
riot
ripple
risk
ritual
rival
river
road
roast
robot
robust
rocket
romance
roof
rookie
room
rose
rotate
rough
round
route
royal
rubber
rude
rug
rule
run
runway
rural
sad
saddle
sadness
safe
sail
salad
salmon
salon
salt
salute
same
sample
sand
satisfy
satoshi
sauce
sausage
save
say
scale
scan
scare
scatter
scene
scheme
school
science
scissors
scorpion
scout
scrap
screen
script
scrub
sea
search
season
seat
second
secret
section
security
seed
seek
segment
select
sell
seminar
senior
sense
sentence
series
service
session
settle
setup
seven
shadow
shaft
shallow
share
shed
shell
sheriff
shield
shift
shine
ship
shiver
shock
shoe
shoot
shop
short
shoulder
shove
shrimp
shrug
shuffle
shy
sibling
sick
side
siege
sight
sign
silent
silk
silly
silver
similar
simple
since
sing
siren
sister
situate
six
size
skate
sketch
ski
skill
skin
skirt
skull
slab
slam
sleep
slender
slice
slide
slight
slim
slogan
slot
slow
slush
small
smart
smile
smoke
smooth
snack
snake
snap
sniff
snow
soap
soccer
social
sock
soda
soft
solar
soldier
solid
solution
solve
someone
song
soon
sorry
sort
soul
sound
soup
source
south
space
spare
spatial
spawn
speak
special
speed
spell
spend
sphere
spice
spider
spike
spin
spirit
split
spoil
sponsor
spoon
sport
spot
spray
spread
spring
spy
square
squeeze
squirrel
stable
stadium
staff
stage
stairs
stamp
stand
start
state
stay
steak
steel
stem
step
stereo
stick
still
sting
stock
stomach
stone
stool
story
stove
strategy
street
strike
strong
struggle
student
stuff
stumble
style
subject
submit
subway
success
such
sudden
suffer
sugar
suggest
suit
summer
sun
sunny
sunset
super
supply
supreme
sure
surface
surge
surprise
surround
survey
suspect
sustain
swallow
swamp
swap
swarm
swear
sweet
swift
swim
swing
switch
sword
symbol
symptom
syrup
system
table
tackle
tag
tail
talent
talk
tank
tape
target
task
taste
tattoo
taxi
teach
team
tell
ten
tenant
tennis
tent
term
test
text
thank
that
theme
then
theory
there
they
thing
this
thought
three
thrive
throw
thumb
thunder
ticket
tide
tiger
tilt
timber
time
tiny
tip
tired
tissue
title
toast
tobacco
today
toddler
toe
together
toilet
token
tomato
tomorrow
tone
tongue
tonight
tool
tooth
top
topic
topple
torch
tornado
tortoise
toss
total
tourist
toward
tower
town
toy
track
trade
traffic
tragic
train
transfer
trap
trash
travel
tray
treat
tree
trend
trial
tribe
trick
trigger
trim
trip
trophy
trouble
truck
true
truly
trumpet
trust
truth
try
tube
tuition
tumble
tuna
tunnel
turkey
turn
turtle
twelve
twenty
twice
twin
twist
two
type
typical
ugly
umbrella
unable
unaware
uncle
uncover
under
undo
unfair
unfold
unhappy
uniform
unique
unit
universe
unknown
unlock
until
unusual
unveil
update
upgrade
uphold
upon
upper
upset
urban
urge
usage
use
used
useful
useless
usual
utility
vacant
vacuum
vague
valid
valley
valve
van
vanish
vapor
various
vast
vault
vehicle
velvet
vendor
venture
venue
verb
verify
version
very
vessel
veteran
viable
vibrant
vicious
victory
video
view
village
vintage
violin
virtual
virus
visa
visit
visual
vital
vivid
vocal
voice
void
volcano
volume
vote
voyage
wage
wagon
wait
walk
wall
walnut
want
warfare
warm
warrior
wash
wasp
waste
water
wave
way
wealth
weapon
wear
weasel
weather
web
wedding
weekend
weird
welcome
west
wet
whale
what
wheat
wheel
when
where
whip
whisper
wide
width
wife
wild
will
win
window
wine
wing
wink
winner
winter
wire
wisdom
wise
wish
witness
wolf
woman
wonder
wood
wool
word
work
world
worry
worth
wrap
wreck
wrestle
wrist
write
wrong
yard
year
yellow
you
young
youth
zebra
zero
zone
zoo
```

## 摘要
![Mnemonic Seed-6](img/Mnemonic%20Seed-6%20(1).png)

## 代码
以下代码片段需要这个wordlist.txt文件。
### Ruby:
```ruby
require 'securerandom'
require 'digest'
require 'openssl'

# -------------------
# 1. Generate Entropy
# -------------------
bytes = SecureRandom.random_bytes(16) # 16 bytes = 128 bits
entropy = bytes.unpack("B*").join
puts "entropy: #{entropy}"

# ----------------------
# 2. Entropy to Mnemonic
# ----------------------
# 1. Create checksum
size = entropy.length / 32 # number of bits to take from hash of entropy (1 bit checksum for every 32 bits entropy)
sha256 = Digest::SHA256.digest([entropy].pack("B*")) # hash of entropy (in raw binary)
checksum = sha256.unpack("B*").join[0..size-1] # get desired number of bits
puts "checksum: #{checksum}"

# 2. Combine
full = entropy + checksum
puts "combined: #{full}"

# 3. Split in to strings of of 11 bits
pieces = full.scan(/.{11}/)

# 4. Get the wordlist as an array
wordlist = File.readlines("wordlist.txt")

# 5. Convert groups of bits to array of words
puts "words:"
sentence = []
pieces.each do |piece|
  i = piece.to_i(2)   # convert string of 11 bits to an integer
  word = wordlist[i]  # get the corresponding word from wordlist
  sentence << word.chomp # add to sentence (removing newline from end of word)
  puts "#{piece} #{i.to_s.rjust(4)} #{word}"
end

mnemonic = sentence.join(" ")
puts "mnemonic: #{mnemonic}"

# -------------------
# 3. Mnemonic to Seed
# -------------------
passphrase = "" # can leave this blank
puts "passphrase: #{passphrase}"

password = mnemonic
salt = "mnemonic#{passphrase}" # "mnemonic" is always used in the salt with optional passphrase appended to it
iterations = 2048
keylength = 64
digest = OpenSSL::Digest::SHA512.new

result = OpenSSL::PKCS5.pbkdf2_hmac(password, salt, iterations, keylength, digest) # password, salt, iter, keylen, digest
seed = result.unpack("H*")[0] # convert to hexadecimal string
puts "seed: #{seed}"
```

### PHP:
```php
<?php

// -------------------
// 1. Generate Entropy
// -------------------

// a. Use Own Entropy (Binary)
// $entropy = "0110101001101101101110100111010101010010010111010111110111011101010011110101100011010001111011110100111001100000001101000101101110110000010010001110101101011011101010000010111101100000100101100000111000101000110000001111010010100000101011000000111010001110";

// b. Use Own Entropy (Hexadecimal)
// $hex = "c0ba5a8e914111210f2bd131f3d5e08d";
// $raw = hex2bin($hex); // convert to raw bytes
// $length = (strlen($raw)/2)*8*2; // calculate expected number of bits in binary representation (so we can pad result to correct number of bits)
// $entropy = str_pad(gmp_strval(gmp_init(bin2hex($raw), 16), 2), $length, "0", STR_PAD_LEFT);
// echo "entropy: $entropy".PHP_EOL;

// c. Generate Entropy
$length  = 256; // How many bits of entropy you want
$urandom = fopen('/dev/urandom', 'r');
$bytes   = fread($urandom, $length/8);  // Get raw bytes (8 bits in a byte)
$entropy = str_pad(gmp_strval(gmp_init(bin2hex($bytes), 16), 2), $length, "0", STR_PAD_LEFT); // Convert to binary (10110111...) (pad zeros)
echo "entropy: $entropy".PHP_EOL;

// ----------------------
// 2. Entropy to Mnemonic
// ----------------------

// 1. Create Checksum
$size = strlen($entropy) / 32; // 1 bit of checksum for every 32 bits of entropy
$hex = str_pad(gmp_strval(gmp_init($entropy, 2), 16), strlen($entropy)/8*2, "0", STR_PAD_LEFT); // convert entropy back to hexadecimal ready to be hashed
$hash = hash("sha256", hex2bin($hex)); // hash raw binary of entropy (bin > hex > rawbinary)
$checksum = substr(str_pad(gmp_strval(gmp_init($hash, 16), 2), 256, "0", STR_PAD_LEFT), 0, $size); // hex > dec > bin, pad with zeros, take size bits
echo "checksum: $checksum".PHP_EOL;

// 2. Combine and split in to groups of 11 bits (32 bits + 1 bit entropy will always be a multiple of 11)
$pieces = str_split($entropy.$checksum, 11);

// 3. Get the wordlist as an array
$wordlist = file("wordlist.txt", FILE_IGNORE_NEW_LINES); // file() reads file in to an array

// 4. Convert groups of 11 bits in to decimal and store the corresponding word
$words = [];
foreach ($pieces as $piece) {
    $i = bindec($piece); // e.g. 01101010011 => 851
    $words[] = $wordlist[$i];
}
$mnemonic = implode(" ", $words); // Convert to a sentence (string of words separated by spaces)
echo "mnemonic: $mnemonic".PHP_EOL;

// -------------------
// 3. Mnemonic to Seed
// -------------------
// Password-Based Key Derivation Function 2

// Password   = mnemonic sentence (UTF-8 NFKD)
// Salt       = "mnemonic" + passphrase (UTF-8 NFKD)
// Iterations = 2048 and HMAC-SHA512
// Algorithm  = HMAC-SHA512 (the pseudo-random function)
// Length     = 512 bits (64 bytes)

$passphrase = "TREZOR"; // default is empty string
echo "passphrase: $passphrase".PHP_EOL;

$seed = bin2hex(hash_pbkdf2("sha512", $mnemonic, "mnemonic".$passphrase, 2048, 64, true)); // algo, password, salt, iterations, length, raw_output
echo "seed: $seed".PHP_EOL;
```

### GO:
```go
package main

import (
    "crypto/rand"   // generating random bytes for entropy
    "crypto/sha256" // creating checksum
    "crypto/sha512" // hash function used in pbkdf2
    "encoding/hex"  // getting bytes from hex string
    "fmt"
    "io/ioutil" // read wordlist file
    "math/big"  // converting byte slice to integer
    "strings"   // split wordlist file string in to an array of strings

    "golang.org/x/crypto/pbkdf2"
)

func main() {

    // ---------------------------------------
    // 1a. Generate Random Bytes (for Entropy)
    // ---------------------------------------

    // Create empty byte slice and fill with bytes from crypto/rand
    bytes := make([]byte, 16) // [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
    rand.Read(bytes)
    fmt.Println("Entropy:", hex.EncodeToString(bytes))
    fmt.Println("Entropy:", bytes)
    fmt.Printf("Entropy: %08b\n", bytes)
    fmt.Println()

    // ----------------------------------------
    // 1b. Use Hexadecimal String (for Entropy)
    // ----------------------------------------

    // entropy := "7f7f7f7f7f7f7f7f7f7f7f7f7f7f7f7f"
    // bytes, _ := hex.DecodeString(entropy)
    // fmt.Println("Entropy:", entropy)
    // fmt.Println("Entropy:", bytes)
    // fmt.Printf("Entropy: %08b\n", bytes)
    // fmt.Println()

    // ------------------
    // 2. Create Checksum
    // ------------------
    // SHA256 the entropy
    h := sha256.New()                        // hash function
    h.Write(bytes)                           // write bytes to hash function
    checksum := h.Sum(nil)                   // get result as a byte slice
    fmt.Printf("Checksum: %08b\n", checksum) // 10000101

    // Get specific number of bits from the checksum
    size := len(bytes) * 8 / 32       // 1 bit for every 32 bits of entropy (1 byte = 8 bits)
    fmt.Println("Bits Wanted:", size) // how many bits from the checksum we want
    fmt.Println()

    // Convert the byte slice to a big integer (so you can use arithmetic to add bits to the end)
    // Note: You can only add bytes (and not individual bits) to a byte slice, so you need to do bitwise arithmetic instead
    //
    //  entropy ([]byte)
    //     |
    //  762603471227441019646259032069709348664 (big.Int)
    //
    dataBigInt := new(big.Int).SetBytes(bytes)
    fmt.Println("Entropy:         ", dataBigInt)

    // Run through the number of bits you want from the checksum, manually adding each bit to the entropy (through arithmetic)
    for i := uint8(0); i < uint8(size); i++ {
        // Add a zero bit to the end for every bit of checksum we add
        //
        //          --->
        //          01001101
        // |entropy|0|
        //
        dataBigInt.Mul(dataBigInt, big.NewInt(2)) // multiplying an integer by two is like adding a 0 bit to the end

        // Use bitwise AND mask to check if each bit of the checksum is set
        //
        // checksum[0] = 01001101
        //           AND 10000000 = 0
        //           AND  1000000 = 1000000
        //           AND   100000 = 0
        //           AND    10000 = 0
        //
        mask := 1 << (7 - i)
        set := uint8(checksum[0]) & uint8(mask) // e.g. 100100100 AND 10000000 = 10000000

        if set > 0 {
            // If the bit is set, change the last zero bit to a 1 bit
            //          10001101
            // |entropy|1|
            //
            dataBigInt.Or(dataBigInt, big.NewInt(1)) // Use bitwise OR to toggle last bit (basically adds 1 to the integer)
        }
    }

    fmt.Println("Entropy+Checksum:", dataBigInt)
    fmt.Println()

    // TIP: Adding bits manually using bitwise arithemtic (can't use bitwise AND or SHIFT on an integer, but maths can do the same)
    // dataBigInt.Mul(dataBigInt, big.NewInt(2)) // add 0 bit to end         = 0  (multiplying by two is like adding a 0 bit)
    // dataBigInt.Or(dataBigInt, big.NewInt(1))  // change 0 bit on end to 1 = 1
    // dataBigInt.Mul(dataBigInt, big.NewInt(2)) // add 0 bit to end         = 10
    // dataBigInt.Mul(dataBigInt, big.NewInt(2)) // add 0 bit to end         = 100
    // dataBigInt.Mul(dataBigInt, big.NewInt(2)) // add 0 bit to end         = 1000

    // ---------------------------------------
    // 3. Convert Entropy+Checksum to Mnemonic
    // ---------------------------------------

    // Get array of words from wordlist file
    file, _ := ioutil.ReadFile("wordlist.txt")
    wordlist := strings.Split(string(file), "\n")
    // fmt.Println(wordlist[2047])

    // Work out expected number of word in mnemonic based on how many 11-bit groups there are in entropy+checksum
    pieces := ((len(bytes) * 8) + size) / 11

    // Create an array of strings to hold words
    words := make([]string, pieces)

    // Loop through every 11 bits of entropy+checksum and convert to corresponding word from wordlist
    for i := pieces - 1; i >= 0; i-- {

        // Use bit mask (bitwise AND) to split integer in to 11-bit pieces
        //
        //            right to left          big.NewInt(2047) = bit mask
        //          <----------------          <--------->
        // 11111111111|11111111111|11111111111|11111111111
        //
        word := big.NewInt(0)                  // hold result of 11 bit mask
        word.And(dataBigInt, big.NewInt(2047)) // bit mask last 11 bits (2047 = 0b11111111111)

        // Add corresponding word to array
        //
        // 11100111000 = 1848 = train
        //
        words[i] = wordlist[word.Int64()] // insert word from wordlist in to array (need to convert big.Int to int64)

        // Remove those 11 bits from end of big integer by bit shifting
        //
        // 11111111111|11111111111|11111111111|11111111111
        //                                    /            - dividing is the same as bit shifting
        //                                    100000000000 = big.NewInt(2048)
        // 11111111111|11111111111|11111111111|
        //
        dataBigInt.Div(dataBigInt, big.NewInt(2048)) // dividing by 2048 is the same as bit shifting 11 bits

    }

    // Convert array of words in to mnemonic string
    mnemonic := strings.Join(words, " ")
    fmt.Println("Mnemonic:", mnemonic)
    fmt.Println()

    // ------------------------------------
    // 4. Convert Mnemonic Sentence to Seed
    // ------------------------------------
    passphrase := "TREZOR"
    seed := pbkdf2.Key([]byte(mnemonic), []byte("mnemonic"+passphrase), 2048, 64, sha512.New)
    fmt.Println("Seed:", hex.EncodeToString(seed))

}

// Code inspired by: https://github.com/tyler-smith/go-bip39/blob/master/bip39.go
```

## 链接
* [BIP 39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)（[Marek Palatinus](https://github.com/slush0)，[Pavol Rusnak](https://rusnak.io/)，[Aaron Voisine](https://github.com/voisine)，Sean Bowe）
* [Ian Coleman BIP39 Tool](https://iancoleman.io/bip39/)（非常棒）

* [Ruby BIP39](https://github.com/sreekanthgs/bip_mnemonic/blob/master/lib/bip_mnemonic.rb)
* [Tyler Smith Go BIP39](https://github.com/tyler-smith/go-bip39)
* [Pete Corey Elixir BIP39](http://www.petecorey.com/blog/2018/02/19/from-bytes-to-mnemonic-using-elixir/)（Elixir是处理二进制的好语言）

* https://en.bitcoin.it/wiki/Seed_phrase


[^1]:https://www.freecodecamp.org/news/how-to-generate-your-very-own-bitcoin-private-key-7ad0f4936e6c/#cryptographically-strong-rng↩︎