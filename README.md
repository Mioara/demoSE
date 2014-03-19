
#-------------------------------------------------------------------------------
#Nom: Kakuro
# But:
# Auteur: Grecu Mioara
#
# Cree: 15/03/2014
# Copyright: (c) 2014 myo
# Licence: <your licence>






import os, sys, types, time, copy
import pygame
import random

PERMUTATION_THRESHOLD_INCREMENT = 40
WATCH_THE_LOGIC_MODE = False

puzzle1 = [\
[-1,-1,[-1,23],[-1,21],-1,-1],\
[-1,[8,15],0,0,-1,-1],\
[[8,-1],0,0,0,-1,-1],\
[[27,-1],0,0,0,0,-1],\
[[5,-1],0,0,-1,-1,-1],\
[[14,-1],0,0,-1,-1,-1],\
[-1,-1,-1,-1,-1,-1]]

puzzle2 = [\
[-1,-1,[-1,31],[-1,4],-1,[-1,23],[-1,4],-1,-1,[-1,19],[-1,5],-1,[-1,29],[-1,11]],\
[-1,[3,15],0,0,[12,3],0,0,[-1,34],[9,-1],0,0,[3,-1],0,0],\
[[31,-1],0,0,0,0,0,0,0,[13,4],0,0,[12,6],0,0],\
[[14,-1],0,0,[9,13],0,0,[9,14],0,0,0,[3,-1],0,0,-1],\
[-1,[15,-1],0,0,-1,[20,19],0,0,0,[-1,18],[11,8],0,0,[-1,13]],\
[-1,[8,8],0,0,[15,9],0,0,0,[12,12],0,0,[14,11],0,0],\
[[3,-1],0,0,[14,-1],0,0,[31,-1],0,0,0,0,0,0,0],\
[[10,-1],0,0,[11,-1],0,0,-1,[17,-1],0,0,[14,-1],0,0,-1]]

puzzle3 = [\
[-1,-1,[-1,40],[-1,16],-1,-1,[-1,44],[-1,3]],\
[-1,[13,30],0,0,[-1,4],[8,-1],0,0],\
[[19,-1],0,0,0,0,[4,3],0,0],\
[[16,-1],0,0,[12,-1],0,0,0,[-1,11]],\
[[12,-1],0,0,[-1,17],[8,-1],0,0,0],\
[[18,-1],0,0,0,[-1,16],[12,-1],0,0],\
[-1,[18,17],0,0,0,[6,16],0,0],\
[[15,-1],0,0,[23,-1],0,0,0,0],\
[[17,-1],0,0,-1,[16,-1],0,0,-1]]

randomkakuro=random.randint(1, 3)
if randomkakuro == 1:
   thePuzzle=puzzle1
elif randomkakuro == 2:
   thePuzzle=puzzle2
elif randomkakuro == 3:
   thePuzzle=puzzle3
else:
   print "4 -Une valeur incorrecte a ?t? g?n?r?"


#thePuzzle = puzzle1

solutionDict = {}
oldSolutionDict = {}

# Cette table de correspondance permet d'?liminer rapidement de nombreux num?ros
# Dans la grille de jeu.
#
# G?n?ration il ne prend pas trop de temps, mais je pense qu'il vaut mieux
# Juste enregistrer les donn?es.
#
# Le premier num?ro de la cl? est le nombre de bo?tes, le deuxi?me
# Est le signe. Z?ros signifient tous les chiffres est possible.
#
# BOGUE: ce tableau est erron?. Voir valeur (3,24) par exemple.
#
numberDict = {\
(2, 3) : [1, 2] ,\
(2, 4) : [1, 3] ,\
(2, 5) : [1, 2, 3, 4] ,\
(2, 6) : [1, 2, 4, 5] ,\
(2, 7) : [1, 2, 3, 4, 5, 6] ,\
(2, 8) : [1, 2, 3, 5, 6, 7] ,\
(2, 9) : [1, 2, 3, 4, 5, 6, 7, 8] ,\
(2, 10) : [1, 2, 3, 4, 6, 7, 8, 9] ,\
(2, 11) : [2, 3, 4, 5, 6, 7, 8, 9] ,\
(2, 12) : [3, 4, 5, 7, 8, 9] ,\
(2, 13) : [4, 5, 6, 7, 8, 9] ,\
(2, 14) : [5, 6, 8, 9] ,\
(2, 15) : [6, 7, 8, 9] ,\
(2, 16) : [7, 9] ,\
(2, 17) : [8, 9] ,\
(3, 6) : [1, 2, 3] ,\
(3, 7) : [1, 2, 4] ,\
(3, 8) : [1, 2, 3, 4, 5] ,\
(3, 9) : [1, 2, 3, 4, 5, 6] ,\
(3, 10) : [1, 2, 3, 4, 5, 6, 7] ,\
(3, 11) : [1, 2, 3, 4, 5, 6, 7, 8] ,\
(3, 12) : 0,\
(3, 13) : 0,\
(3, 14) : 0,\
(3, 15) : 0,\
(3, 16) : 0,\
(3, 17) : 0,\
(3, 18) : 0,\
(3, 19) : [2, 3, 4, 5, 6, 7, 8, 9] ,\
(3, 20) : [3, 4, 5, 6, 7, 8, 9] ,\
(3, 21) : [4, 5, 6, 7, 8, 9] ,\
(3, 22) : [5, 6, 7, 8, 9] ,\
(3, 23) : [6, 8, 9] ,\
(3, 24) : [7, 8, 9] ,\
(4, 10) : [1, 2, 3, 4] ,\
(4, 11) : [1, 2, 3, 5] ,\
(4, 12) : [1, 2, 3, 4, 5, 6] ,\
(4, 13) : [1, 2, 3, 4, 5, 6, 7] ,\
(4, 14) : [1, 2, 3, 4, 5, 6, 7, 8] ,\
(4, 15) : 0,\
(4, 16) : 0,\
(4, 17) : 0,\
(4, 18) : 0,\
(4, 19) : 0,\
(4, 20) : 0,\
(4, 21) : 0,\
(4, 22) : 0,\
(4, 23) : 0,\
(4, 24) : 0,\
(4, 25) : 0,\
(4, 26) : [2, 3, 4, 5, 6, 7, 8, 9] ,\
(4, 27) : [3, 4, 5, 6, 7, 8, 9] ,\
(4, 28) : [4, 5, 6, 7, 8, 9] ,\
(4, 29) : [5, 7, 8, 9] ,\
(4, 30) : [6, 7, 8, 9] ,\
(5, 15) : [1, 2, 3, 4, 5] ,\
(5, 16) : [1, 2, 3, 4, 6] ,\
(5, 17) : [1, 2, 3, 4, 5, 6, 7] ,\
(5, 18) : [1, 2, 3, 4, 5, 6, 7, 8] ,\
(5, 19) : 0,\
(5, 20) : 0,\
(5, 21) : 0,\
(5, 22) : 0,\
(5, 23) : 0,\
(5, 24) : 0,\
(5, 25) : 0,\
(5, 26) : 0,\
(5, 27) : 0,\
(5, 28) : 0,\
(5, 29) : 0,\
(5, 30) : 0,\
(5, 31) : 0,\
(5, 32) : [2, 3, 4, 5, 6, 7, 8, 9] ,\
(5, 33) : [3, 4, 5, 6, 7, 8, 9] ,\
(5, 34) : [4, 6, 7, 8, 9] ,\
(5, 35) : [5, 6, 7, 8, 9] ,\
(6, 21) : [1, 2, 3, 4, 5, 6] ,\
(6, 22) : [1, 2, 3, 4, 5, 7] ,\
(6, 23) : [1, 2, 3, 4, 5, 6, 7, 8] ,\
(6, 24) : 0,\
(6, 25) : 0,\
(6, 26) : 0,\
(6, 27) : 0,\
(6, 28) : 0,\
(6, 29) : 0,\
(6, 30) : 0,\
(6, 31) : 0,\
(6, 32) : 0,\
(6, 33) : 0,\
(6, 34) : 0,\
(6, 35) : 0,\
(6, 36) : 0,\
(6, 37) : [2, 3, 4, 5, 6, 7, 8, 9] ,\
(6, 38) : [3, 5, 6, 7, 8, 9] ,\
(6, 39) : [4, 5, 6, 7, 8, 9] ,\
(7, 28) : [1, 2, 3, 4, 5, 6, 7] ,\
(7, 29) : [1, 2, 3, 4, 5, 6, 8] ,\
(7, 30) : 0,\
(7, 31) : 0,\
(7, 32) : 0,\
(7, 33) : 0,\
(7, 34) : 0,\
(7, 35) : 0,\
(7, 36) : 0,\
(7, 37) : 0,\
(7, 38) : 0,\
(7, 39) : 0,\
(7, 40) : 0,\
(7, 41) : [2, 4, 5, 6, 7, 8, 9] ,\
(7, 42) : [3, 4, 5, 6, 7, 8, 9] ,\
(8, 36) : [1, 2, 3, 4, 5, 6, 7, 8] ,\
(8, 37) : [1, 2, 3, 4, 5, 6, 7, 9] ,\
(8, 38) : [1, 2, 3, 4, 5, 6, 8, 9] ,\
(8, 39) : [1, 2, 3, 4, 5, 7, 8, 9] ,\
(8, 40) : [1, 2, 3, 4, 6, 7, 8, 9] ,\
(8, 41) : [1, 2, 3, 5, 6, 7, 8, 9] ,\
(8, 42) : [1, 2, 4, 5, 6, 7, 8, 9] ,\
(8, 43) : [1, 3, 4, 5, 6, 7, 8, 9] ,\
(8, 44) : [2, 3, 4, 5, 6, 7, 8, 9] ,\
}


puzzleRows = len(thePuzzle)
puzzleCols = len(thePuzzle[0])

# Adjust this (and only this) to change the size of the window
# It can only be square for now (simplifies scaling the numbers)
scalefactor = 65

height = puzzleRows * scalefactor
width = puzzleCols * scalefactor

# Cette fonction retourne un rectangle d'une cellule particuli?re sur le plateau de jeu.
def rectForXYcell (x, y):
    return pygame.Rect(width/puzzleCols*y,height/puzzleRows*x,width/puzzleCols+1,height/puzzleRows+1)

# Cette fonction d?termine toutes les combinaisons possibles des listes adopt?es.
# Par exemple, si vous passez [[2,4], [4,3]], cette fonction serait de retour
# [[2,7], [2,3], [4,7], [4,3].
#
# Aucun doublon n'est autoris? dans les s?ries, alors si vous passez [[2,3], [2,4]]
# Il reviendra [[2,4], [3,2], [3,4]]. Le [2,2] article sera omis.
#
def permutate(array):
	numOfPermutations = 1
	for i in range(len(array)):
		numOfPermutations = numOfPermutations * len(array[i])
	theIndex = [0]*len(array)
	theList = []
	for x in range(numOfPermutations):
		subList = []
		for i in range(len(array)):
			subList.append(array[i][theIndex[i]])
		if len(set(subList)) == len(subList):
			theList.append(subList)
		for i in range(len(array)):
			theIndex[i] = theIndex[i] + 1
			if theIndex[i] != len(array[i]):
				break
			else:
				theIndex[i] = 0
	return theList

# Cette fonction attend une pression sur une touche, et retourne le contr?le
# ? la fonction de parent quand on est re?u.
def waitForKeypress():
	while 1:
		event = pygame.event.wait()
		if event.type == pygame.QUIT:
			sys.exit()
		if event.type == pygame.KEYDOWN:
			return
# Cette fonction it?ratt? sur les cellules, les permutations de contr?le
# Pour trouver ceux qui s'ajoutent ? l'indicateur. Il met ensuite ? jour le
# Matrice de solution avec les nouvelles donn?es, en ?liminant tous les num?ros
# Qui ?taient dans la matrice de la solution, mais n'?tait pas valable dans toute
# Permutations. Il est assez lent.
#
# Cette fonction devra ?tre appel?e plusieurs fois
# Avant il va compl?tement optimiser la matrice.
#
# Index doit ?tre une valeur indiquant combien de fois cette fonction
# A d?j? ex?cut?. Le plus grand indice est ?lev?, plus tol?rante ce
# Fonction sera de grandes permutations. En sautant ces grands
# Permutations d'abord et en se concentrant sur ceux ?faciles?, nous
# R?duisent la quantit? totale de calcul n?cessaire pour r?soudre le puzzle.
#
# TODO: il ya trop de code en double dans le "horizontalement" et
# Sections ?verticalement?. Il devrait ?tre fix?.
#


def removeIllegalValues(index):
	lastHint = nextHint = 0
	arrayIndexes = []
	# Horizontale
	statusBarMessage("V?rification de permutations par ligne. Ignorer les lignes > %d." % PERMUTATION_THRESHOLD_INCREMENT*index)
	for x in range(puzzleRows):
		for y in range(puzzleCols):
			if isList(thePuzzle[x][y]) and thePuzzle[x][y][0]!=-1:
				nextHint = thePuzzle[x][y][0]
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				arrayIndexes.append(thePuzzle[x][y])
			if (thePuzzle[x][y] == -1 or isList(thePuzzle[x][y])) and lastHint != 0:
				theBlock = []
				for theIndex in arrayIndexes:
					theBlock.append(list(solutionDict[theIndex]))
				numOfPermutations = 1
				for i in range(len(theBlock)):
					numOfPermutations = numOfPermutations * len(theBlock[i])
				if numOfPermutations < PERMUTATION_THRESHOLD_INCREMENT*index:
					mutations = permutate(theBlock)
					goodMutations = []
					for theMutation in mutations:
						if sum(theMutation) == lastHint:
							goodMutations.append(theMutation)
					if goodMutations == []:
						print "Puzzle incoh?rentes."
						print "block %dx%d" %(x,y)
						sys.exit()
					for t in range(len(arrayIndexes)):
						theBlock = []
						for s in range(len(goodMutations)):
							theBlock.append(goodMutations[s][t])
						solutionDict[arrayIndexes[t]] = set(theBlock)
						if WATCH_THE_LOGIC_MODE:
							redrawAllWhiteBoxes()
							waitForKeypress()
				arrayIndexes = []
				lastHint = 0
			if nextHint:
				lastHint = nextHint
				nextHint = 0
	statusBarMessage("V?rification de permutations par colonne. ignorant colonnes > %d." % PERMUTATION_THRESHOLD_INCREMENT*index)
	# Verticale
	for y in range(puzzleCols):
		for x in range(puzzleRows):
			if isList(thePuzzle[x][y]) and thePuzzle[x][y][1]!=-1:
				nextHint = thePuzzle[x][y][1]
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				arrayIndexes.append(thePuzzle[x][y])
			if (thePuzzle[x][y] == -1 or isList(thePuzzle[x][y])) and lastHint != 0:
				theBlock = []
				for i in range(len(arrayIndexes)):
					theBlock.append(list(solutionDict[arrayIndexes[i]]))
				numOfPermutations = 1
				for i in range(len(theBlock)):
					numOfPermutations = numOfPermutations * len(theBlock[i])
				if numOfPermutations < PERMUTATION_THRESHOLD_INCREMENT*index:
					mutations = permutate(theBlock)
					goodMutations = []
					for theMutation in mutations:
						if sum(theMutation) == lastHint:
							goodMutations.append(theMutation)
					if goodMutations == []:
						print "Puzzle incoh?rentes."
						print "block %dx%d" %(x,y)
						print theBlock
						print lastHint
						print mutations
						sys.exit()
					for t in range(len(arrayIndexes)):
						theBlock = []
						for s in range(len(goodMutations)):
							theBlock.append(goodMutations[s][t])
						solutionDict[arrayIndexes[t]] = set(theBlock)
						if WATCH_THE_LOGIC_MODE:
							redrawAllWhiteBoxes()
							waitForKeypress()
				arrayIndexes = []
				lastHint = 0
			if nextHint:
				lastHint = nextHint
				nextHint = 0

# Si des cases sur le plateau ont ?t? r?solus, c'est-?-r?duite
# Pour une r?ponse possible, alors que la r?ponse peut ?tre enlev? comme un
# Possibilit? de toutes les autres cellules en ligne avec cette cellule. que
# C'est exactement ce que cette fonction fait.
#
# Cette fonction peut cr?er des cellules suppl?mentaires qui n'ont qu'un seul
# Entr?e, donc il doit ?tre ex?cut? plusieurs fois pour assurer la
# Carte est enti?rement optimis?.
#
# TODO: il ya trop de code en double dans le "horizontalement" et
# Sections ?verticalement?. Il devrait ?tre fix?.
#
# TODO: D?tection de savoir si cette fonction doit ?tre ex?cut?e
# Doit ?tre ajout?e ? d'autres fonctions d'?limination
#
def removeDuplicateValues():
	lastHint = nextHint = 0
	arrayIndexes = []
	# Horizontale
	statusBarMessage("Retrait Valeurs dupliqu?es par les ligne.")
	for x in range(puzzleRows):
		for y in range(puzzleCols):
			if isList(thePuzzle[x][y]) and thePuzzle[x][y][0]!=-1:
				nextHint = thePuzzle[x][y][0]
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				arrayIndexes.append(thePuzzle[x][y])
			if (thePuzzle[x][y] == -1 or isList(thePuzzle[x][y])) and lastHint != 0:
				for theIndex in arrayIndexes:
					if len(solutionDict[theIndex])==1:
						for theOtherIndex in arrayIndexes:
							if theOtherIndex != theIndex:
								solutionDict[theOtherIndex] = solutionDict[theOtherIndex] - solutionDict[theIndex]
								if WATCH_THE_LOGIC_MODE:
									redrawAllWhiteBoxes()
									waitForKeypress()
				lastHint = 0
				arrayIndexes = []
			if nextHint:
				lastHint = nextHint
				nextHint = 0
	# Verticale
	statusBarMessage("Retrait Valeurs dupliqu?es les collone par.")
	for y in range(puzzleCols):
		for x in range(puzzleRows):
			if isList(thePuzzle[x][y]) and thePuzzle[x][y][1]!=-1:
				nextHint = thePuzzle[x][y][1]
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				arrayIndexes.append(thePuzzle[x][y])
			if (thePuzzle[x][y] == -1 or isList(thePuzzle[x][y])) and lastHint != 0:
				for theIndex in arrayIndexes:
					if len(solutionDict[theIndex])==1:
						for theOtherIndex in arrayIndexes:
							if theOtherIndex != theIndex:
								solutionDict[theOtherIndex] = solutionDict[theOtherIndex] - solutionDict[theIndex]
								if WATCH_THE_LOGIC_MODE:
									redrawAllWhiteBoxes()
									waitForKeypress()
				lastHint = 0
				arrayIndexes = []
			if nextHint:
				lastHint = nextHint
				nextHint = 0
# Cette fonction retourne vrai si le puzzle a ?t? r?solu,
# False si elle n'a pas ?t?.
#
# TODO: il ya trop de code en double dans le "horizontalement" et
# Sections ?verticalement?. Il devrait ?tre fix?.
#
def isPuzzleSolved():
	# Horizontale
	for x in range(puzzleRows):
		for y in range(puzzleCols):
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				if len(solutionDict[thePuzzle[x][y]]) > 1:
					return 0
	# Verticale
	for y in range(puzzleCols):
		for x in range(puzzleRows):
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				if len(solutionDict[thePuzzle[x][y]]) > 1:
					return 0
	return 1

# Cette fonction initialise la solution ?limination dictionnaire avec
# Les valeurs par d?faut, c'est-?-1 ? 9 pour chaque carr?. Il a ?galement
# Remplace les z?ros dans la matrice de puzzle avec des valeurs uniques pour
# Dictionnaire facile recherche.
#
# TODO: il ya trop de code en double dans le "horizontalement" et
# Sections ?verticalement?. Il devrait ?tre fix?.
#
def initSolutionDict():
	uniqueInt = 1
	for x in range(puzzleRows):
		for y in range(puzzleCols):
			if thePuzzle[x][y] == 0:
				solutionDict[uniqueInt] = set([1,2,3,4,5,6,7,8,9])
				thePuzzle[x][y] = uniqueInt
				uniqueInt = uniqueInt + 1

# Cette fonction r?duit l'espace de recherche de la solution pr?vue par les
# En utilisant la table de conversion pr?d?fini en haut de la page.
# Il est rapide et efficace, mais probablement moins utile sur
# Puzzles de type "dur".
#
# Il n'est pas n?cessaire d'ex?cuter cette fonction plus d'une fois.
#
# TODO: il ya trop de code en double dans le "horizontalement" et
# Sections ?verticalement?. Il devrait ?tre fix?.
#
def reduceSearchSpace():
	lastHint = nextHint = 0
	arrayIndexes = []
	statusBarMessage("R?duire ligne de recherche en utilisant l'espace de consultation de table.")
	#Horizontally
	for x in range(puzzleRows):
		for y in range(puzzleCols):
			if isList(thePuzzle[x][y]) and thePuzzle[x][y][0]!=-1:
				nextHint = thePuzzle[x][y][0]
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				arrayIndexes.append(thePuzzle[x][y])
			if (thePuzzle[x][y] == -1 or isList(thePuzzle[x][y])) and lastHint != 0:
				for z in arrayIndexes:
					try:
						entry = numberDict[(len(arrayIndexes),lastHint)]
						if entry != 0:
							solutionDict[z] = set(entry)
							if WATCH_THE_LOGIC_MODE:
								redrawAllWhiteBoxes()
								waitForKeypress()
					except KeyError:
						print 'KeyError (casse-t?te incompatible)'
				arrayIndexes = []
				lastHint = 0
			if nextHint:
				lastHint = nextHint
				nextHint = 0
	# Verticale
	statusBarMessage("R?duire recherche colonne espace en utilisant la table de correspondance.")
	for y in range(puzzleCols):
		for x in range(puzzleRows):
			if isList(thePuzzle[x][y]) and thePuzzle[x][y][1]!=-1:
				nextHint = thePuzzle[x][y][1]
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				arrayIndexes.append(thePuzzle[x][y])
			if (thePuzzle[x][y] == -1 or isList(thePuzzle[x][y])) and lastHint != 0:
				for z in arrayIndexes:
					try:
						entry = numberDict[(len(arrayIndexes),lastHint)]
						if entry != 0:
							solutionDict[z] = solutionDict[z] & set(entry)
							if WATCH_THE_LOGIC_MODE:
								redrawAllWhiteBoxes()
								waitForKeypress()
					except KeyError:
						print 'KeyError (puzzle inconsistent)'
				arrayIndexes = []
				lastHint = 0
			if nextHint:
				lastHint = nextHint
				nextHint = 0

# Cette fonction retrace la case blanche ? l'index x, y et la
# Matrice int?rieur
def redrawWhiteBox(x,y):
	# Necessary?
	screen.fill([255,255,255],rectForXYcell(x,y))
	pygame.draw.rect(screen,[0,0,0],rectForXYcell(x,y),1)

	try:
		possibleSolutions = solutionDict[thePuzzle[x][y]]
		if len(possibleSolutions) == 1:
			# Je ne peux pas imaginer un meilleur fa?on de faire=/
			theDigit = possibleSolutions.pop()
			possibleSolutions.add(theDigit)
			therender = bigfont.render("%d" %theDigit,1,[0,0,255])
			screen.blit(therender,[width/puzzleCols*(y+0.5)-therender.get_width()/2,height/puzzleRows*(x+0.5)-therender.get_height()/2])
			return
		else:
			if(1 in possibleSolutions):
				string1 = "1 "
			else:
				string1 = "  "
			if(2 in possibleSolutions):
				string1 = string1 + "2 "
			else:
				string1 = string1 + "  "
			if(3 in possibleSolutions):
				string1 = string1 + "3"
			else:
				string1 = string1 + " "
			if(4 in possibleSolutions):
				string2 = "4 "
			else:
				string2 = "  "
			if(5 in possibleSolutions):
				string2 = string2 + "5 "
			else:
				string2 = string2 + "  "
			if(6 in possibleSolutions):
				string2 = string2 + "6"
			else:
				string2 = string2 + " "
			if(7 in possibleSolutions):
				string3 = "7 "
			else:
				string3 = "  "
			if(8 in possibleSolutions):
				string3 = string3 + "8 "
			else:
				string3 = string3 + "  "
			if(9 in possibleSolutions):
				string3 = string3 + "9"
			else:
				string3 = string3 + " "
	except KeyError:
		string1 = "1 2 3"
		string2 = "4 5 6"
		string3 = "7 8 9"

	therender = monofont.render(string1,1,[0,128,0])
	screen.blit(therender,[width/puzzleCols*(y+0.5)-therender.get_width()/2,height/puzzleRows*(x+0.2)-therender.get_height()/2])
	therender = monofont.render(string2,1,[0,128,0])
	screen.blit(therender,[width/puzzleCols*(y+0.5)-therender.get_width()/2,height/puzzleRows*(x+0.5)-therender.get_height()/2])
	therender = monofont.render(string3,1,[0,128,0])
	screen.blit(therender,[width/puzzleCols*(y+0.5)-therender.get_width()/2,height/puzzleRows*(x+0.8)-therender.get_height()/2])

# Cette fonction retrace toutes les cases blanches et les matricies int?rieur.
# Il retourne ?galement la surface ? double m?moire tampon d'affichage.
def redrawAllWhiteBoxes():
	for x in range(puzzleRows):
		for y in range(puzzleCols):
			if thePuzzle[x][y] > 0 and not isList(thePuzzle[x][y]):
				redrawWhiteBox(x,y)
				try:
					if oldSolutionDict[thePuzzle[x][y]] != solutionDict[thePuzzle[x][y]]:
						pygame.draw.rect(screen,[255,0,0],rectForXYcell(x,y).inflate(-4,-4),3)
				except KeyError:
					None
# Enregistrer la liste pour comparer la prochaine fois
	oldSolutionDict.update(solutionDict)
	pygame.display.flip()

def isList(l):
	"""M?thode pratique qui fonctionne avec toutes les versions 2.x de Python
	pour d?terminer si oui ou non quelque chose est listlike."""
	return hasattr(l, '__iter__') \
		or (type(l) in (types.ListType, types.TupleType))

if not pygame.font: print 'Attention, les polices sont d?sactiv?es'
if not pygame.mixer: print 'Attention, les polices sont d?sactiv?es'

# Cette fonction imprime une corde ? notre barre d'?tat
def statusBarMessage(theMessage):
	screen.fill([0,0,0],pygame.Rect(0,height,width,30))
	therender = statusfont.render(theMessage,1,[255,255,255],[0,0,0])
	screen.blit(therender,[width/2-therender.get_width()/2,height+15-therender.get_height()/2])

pygame.init()
screen = pygame.display.set_mode([width+1,height+30])
thefont = pygame.font.Font(pygame.font.get_default_font(),scalefactor/3)
monofont = pygame.font.Font(pygame.font.match_font('Courier','Courier New','Monospaced'),scalefactor/3)
bigfont = pygame.font.Font(pygame.font.get_default_font(),scalefactor/1)
statusfont = pygame.font.Font(pygame.font.get_default_font(),16)

if monofont == None:
	print "le fonte n'existe pas"
# Dessiner Le Conseil
for x in range(puzzleRows):
	for y in range(puzzleCols):
		# bo?te noire
		if thePuzzle[x][y] == -1:
			screen.fill([131,139,131],rectForXYcell(x,y))
			pygame.draw.rect(screen,[255,255,255],rectForXYcell(x,y),1)
		# Bo?te noire avec la piste et le nombre
		elif isList(thePuzzle[x][y]):
			screen.fill([131,139,131],rectForXYcell(x,y))
			pygame.draw.rect(screen,[255,255,255],rectForXYcell(x,y),1)
			pygame.draw.line(screen, [255,255,255], [width/puzzleCols*y,height/puzzleRows*x], [width/puzzleCols*(y+1),height/puzzleRows*(x+1)], 1)
			if thePuzzle[x][y][0] != -1:
				therender = thefont.render("%d" %thePuzzle[x][y][0],1,[255,255,255],[131,139,131])
				screen.blit(therender,[width/puzzleCols*(y+0.75)-therender.get_width()/2,height/puzzleRows*(x+0.30)-therender.get_height()/2])
			if thePuzzle[x][y][1] != -1:
				therender = thefont.render("%d" %thePuzzle[x][y][1],1,[255,255,255],[131,139,131])
				screen.blit(therender,[width/puzzleCols*(y+0.25)-therender.get_width()/2,height/puzzleRows*(x+0.75)-therender.get_height()/2])
		# bo?te blanc
		else:
			redrawWhiteBox(x,y)

pygame.display.flip()
waitForKeypress()

# R?soudre le Conseil
startTime = time.clock()
initSolutionDict()
reduceSearchSpace()

duration = time.clock() - startTime
grandStartTime = startTime
print "Recherche r?duction de l'espace achev?e en %.2f secondes." %duration
puzzleSolved = 0
for count in range(100):
	for i in range(2):
		startTime = time.clock()
		removeIllegalValues(count*2-1+i)
		duration = time.clock() - startTime
		print "It?ration ill?gal de valeur achev?e en %.2f secondes." %duration
	for i in range(2):
		startTime = time.clock()
		removeDuplicateValues()
		duration = time.clock() - startTime
		print "It?ration double de valeur achev?e en %.2f secondes." %duration
	if isPuzzleSolved():
		puzzleSolved = 1
		break
duration = time.clock() - grandStartTime
if(puzzleSolved):
	print "Puzzle r?solu en %.3f secondes." %duration
else:
	print "Je ne pouvais pas r?soudre cette ?nigme, mais j'ai essay? de %.3f secondes." %duration

redrawAllWhiteBoxes()
# D?but de la boucle de l'?v?nement                                                                                                                for i in range(len(a)):
while 1:
	event = pygame.event.wait()
	if event.type == pygame.QUIT:
		sys.exit()
	if event.type == 1:
		None


