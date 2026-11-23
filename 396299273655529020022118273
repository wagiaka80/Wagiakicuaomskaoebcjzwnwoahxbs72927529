--========================================================
-- SUNSET UI + OLLIE 99 NIGHTS (COMPLETE & OPTIMIZED)
-- TP Auto Chop (Exact Tree Name Filter + Anti-Lose Axe Fix)
-- Independent Kill Aura + Conflict Warning | Auto Rescue Children
-- Cached Chest + Ammo + Coins | Zero-Lag Mobile Ready
--========================================================

local Players      = game:GetService("Players")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting     = game:GetService("Lighting")
local RS           = game:GetService("ReplicatedStorage")
local Workspace    = game:GetService("Workspace")
local CoreGui      = game:GetService("CoreGui")

local Player    = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Cleanup old GUIs
for _, name in ipairs({"OllieUI", "OllieMenu", "ollie_v9", "SunsetUI"}) do
	pcall(function()
		local g = PlayerGui:FindFirstChild(name)
		if g then g:Destroy() end
	end)
	pcall(function()
		local g = CoreGui:FindFirstChild(name)
		if g then g:Destroy() end
	end)
end

--========================================================
-- REMOTES & GAME DATA
--========================================================

local Remotes          = RS:WaitForChild("RemoteEvents", 10)
local EquipRE          = Remotes and Remotes:FindFirstChild("EquipItemHandle")
local UnequipRE        = Remotes and Remotes:FindFirstChild("UnequipItemHandle")
local DamageRF         = Remotes and Remotes:FindFirstChild("ToolDamageObject")
local MapRE            = Remotes and Remotes:FindFirstChild("MapCellsDiscovered")
local DragRE           = Remotes and Remotes:FindFirstChild("RequestStartDraggingItem")
local StopDragRE       = Remotes and Remotes:FindFirstChild("StopDraggingItem")
local HitSoundRE       = Remotes and Remotes:FindFirstChild("PlayEnemyHitSound")
local ReplicateSoundRE = Remotes and Remotes:FindFirstChild("RequestReplicateSound")
local PlantRE          = Remotes and Remotes:FindFirstChild("RequestPlantItem")
local ConsumeRF        = Remotes and Remotes:FindFirstChild("RequestConsumeItem")
local OpenChestRE      = Remotes and Remotes:FindFirstChild("RequestOpenItemChest")
local CollectCoinRF    = Remotes and Remotes:FindFirstChild("RequestCollectCoints")
local HotbarItemRF     = Remotes and Remotes:FindFirstChild("RequestHotbarItem")
local BagStoreRF       = Remotes and Remotes:FindFirstChild("RequestBagStoreItem")
local BagDropRE        = Remotes and Remotes:FindFirstChild("RequestBagDropItem")

local AXES = {
	"Old Axe","Good Axe","Ice Axe","Strong Axe","Chainsaw","Admin Axe",
}
local HEAVY_AXES = {
	["Strong Axe"] = true,
	["Chainsaw"]   = true,
	["Admin Axe"]  = true,
}
local MELEE_WEAPONS = {
	"Spear","Honey Spear","Morningstar","Katana","Laser Sword","Ice Sword",
	"Trident","Poison Claws","Poison Spear","Infernal Sword",
	"Cultist King Mace","Brutal Battleaxe","Obsidiron Hammer",
	"Scythe","Vampire Scythe","Happy's Scythe",
}
local SACK_NAMES = {
	"Admin Sack","Strong Sack","Good Sack","Old Sack","Sack","Bag",
}
local FOOD_ITEMS  = { "Berry","Carrot","Apple","Corn","Pumpkin","Chili" }
local SCRAP_ITEMS = {
	"Log","Chair","Bolt","Sheet Metal","Broken Fan","Broken Radio",
	"Broken Microwave","Washing Machine","Tires & Engines",
}
local FUEL_EXACT = {
	"Log","Chair","Biofuel","Coal",
	"Purple Fur Tuft","Fuel Canister","Oil Barrel",
}
local CHEST_KEYS = { "Chest", "Crate", "Supply", "Loot" }
local COIN_KEYS  = { "Coin" }
local AMMO_KEYS  = { "Ammo", "Bullet", "Shell", "Arrow", "Dart", "Rocket", "Cartridge" }
local IGNORE_LOOT_KEYS = {
	"Tree","Pine","Oak","Birch","Fir","Bush","Foliage","Wall","Floor","Ground",
	"Terrain","Campfire","MainFire","Spawn","Part","Baseplate","Hitbox"
}

local CFG = {
	Speed=false,            WalkSpeed=40,
	Jump=false,             JumpPower=90,
	Fly=false,              FlySpeed=60,
	Noclip=false,
	KillAura=false,         KillRadius=150,
	AutoChop=false,         ChopRadius=130,
	AutoDrop=false,
	AutoEat=false,
	AutoChestAndLoot=false,
	AutoCollectCoins=false,
	AutoRescueChild=false,
	ESPChild=false,
	Fullbright=false,
	FixLag=false,
	Delay=0.35,
}

local MAX_KILL_PER_CYCLE = 6
local TOOL_SUFFIX = "11960515286"
local CHILD_KEY   = "Lost Child"

--========================================================
-- GAME HELPERS (OPTIMIZED)
--========================================================

local function hasKey(name, keys)
	if type(name) ~= "string" then return false end
	local n = string.lower(name)
	for _, k in ipairs(keys) do
		if string.find(n, string.lower(k), 1, true) then return true end
	end
	return false
end

local function matchesExact(name, list)
	for _, v in ipairs(list) do if name == v then return true end end
	return false
end

local function root()
	local c = Player.Character
	return c and c:FindFirstChild("HumanoidRootPart")
end

local function hum()
	local c = Player.Character
	return c and c:FindFirstChildOfClass("Humanoid")
end

local function getHeadPart()
	local c = Player.Character
	return c and (c:FindFirstChild("Head") or c:FindFirstChild("HumanoidRootPart"))
end

local function partOf(m)
	if not m then return nil end
	if m:IsA("BasePart") then return m end
	if m:IsA("Model") then
		if m.PrimaryPart then return m.PrimaryPart end
		-- Ưu tiên lấy phần thân cây / hitbox nếu có
		for _, child in ipairs(m:GetDescendants()) do
			if child:IsA("BasePart") and hasKey(child.Name, { "Trunk", "Wood", "Log", "Hitbox", "Stem" }) then
				return child
			end
		end
		local hrp = m:FindFirstChild("HumanoidRootPart")
		if hrp then return hrp end
		return m:FindFirstChildWhichIsA("BasePart", true)
	end
	return nil
end

local function findFromList(list, inv, char)
	for _, name in ipairs(list) do
		local t = (inv and inv:FindFirstChild(name))
			or (char and char:FindFirstChild(name))
		if t then return t end
	end
end

local function getFallbackTool(inv, char)
	if inv then
		local t = inv:FindFirstChildWhichIsA("Tool")
		if t then return t end
		for _, item in ipairs(inv:GetChildren()) do
			if item:IsA("Model") then return item end
		end
	end
	return char and char:FindFirstChildWhichIsA("Tool")
end

local function getKillTool()
	local inv, char = Player:FindFirstChild("Inventory"), Player.Character
	return findFromList(MELEE_WEAPONS, inv, char)
		or findFromList(AXES, inv, char)
		or getFallbackTool(inv, char)
end

local function getChopTool()
	local inv, char = Player:FindFirstChild("Inventory"), Player.Character
	local priorityAxes = { "Admin Axe", "Chainsaw", "Strong Axe", "Ice Axe", "Good Axe", "Old Axe" }
	return findFromList(priorityAxes, inv, char)
		or findFromList(AXES, inv, char)
		or findFromList(MELEE_WEAPONS, inv, char)
		or getFallbackTool(inv, char)
end

local function getSackTool()
	local inv, char = Player:FindFirstChild("Inventory"), Player.Character
	local exact = findFromList(SACK_NAMES, inv, char)
	if exact then return exact end
	if inv then
		for _, item in ipairs(inv:GetChildren()) do
			if hasKey(item.Name, { "Sack", "Bag" }) then return item end
		end
	end
	if char then
		for _, item in ipairs(char:GetChildren()) do
			if hasKey(item.Name, { "Sack", "Bag" }) then return item end
		end
	end
	return nil
end

local cachedFirePart = nil
local function getFirePart()
	if cachedFirePart and cachedFirePart.Parent then
		return cachedFirePart
	end
	local map  = Workspace:FindFirstChild("Map")
	local camp = map and map:FindFirstChild("Campground")
	local fire = camp and camp:FindFirstChild("MainFire", true)
	if fire then
		cachedFirePart = partOf(fire)
		return cachedFirePart
	end
	for _, o in ipairs(Workspace:GetDescendants()) do
		if o.Name == "MainFire" or o.Name == "Campfire" then
			local p = partOf(o)
			if p then
				cachedFirePart = p
				return p
			end
		end
	end
	return nil
end

local function getNilByName(name)
	local out = {}
	if not getnilinstances then return out end
	local ok, nilList = pcall(getnilinstances)
	if not ok or not nilList then return out end
	for _, obj in ipairs(nilList) do
		local ok2, n = pcall(function() return obj.Name end)
		if ok2 and n == name then
			table.insert(out, obj)
		end
	end
	return out
end

--========================================================
-- EVENT-BASED CACHE: CHILDREN (NO WORKSPACE LAG)
--========================================================

local childSet = {}
local function isChildModel(o)
	return o and o:IsA("Model") and string.find(o.Name, CHILD_KEY, 1, true) ~= nil
end

local function initChildCache()
	local charFolder = Workspace:FindFirstChild("Characters")
	local scanTargets = { Workspace }
	if charFolder then table.insert(scanTargets, charFolder) end

	for _, folder in ipairs(scanTargets) do
		for _, o in ipairs(folder:GetChildren()) do
			if isChildModel(o) then childSet[o] = true end
		end
		folder.ChildAdded:Connect(function(o)
			if isChildModel(o) then childSet[o] = true end
		end)
		folder.ChildRemoved:Connect(function(o)
			childSet[o] = nil
		end)
	end

	task.spawn(function()
		local all = Workspace:GetDescendants()
		for i = 1, #all do
			local o = all[i]
			if isChildModel(o) then childSet[o] = true end
			if i % 800 == 0 then task.wait() end
		end
	end)
end
initChildCache()

--========================================================
-- AUTO RESCUE CHILDREN (TP -> BAG -> CAMPFIRE -> DROP)
--========================================================

local rescueStatusText   = "Chờ bật tự động cứu trẻ..."
local rescueStatusLbl    = nil
local rescuedChildren    = setmetatable({}, { __mode = "k" })
local rescuedChildKeys   = {}
local totalChildrenSaved = 0

local function updateRescueStatus(txt)
	rescueStatusText = txt
	if rescueStatusLbl and rescueStatusLbl.Parent then
		rescueStatusLbl.Text = txt
	end
end

local function getChildUniqueKey(child)
	if not child then return nil end
	local ok, dbg = pcall(function() return child:GetDebugId() end)
	if ok and dbg then return dbg end
	return tostring(child)
end

local function isChildAlreadyRescued(child)
	if not child then return true end
	if rescuedChildren[child] then return true end
	local key = getChildUniqueKey(child)
	if key and rescuedChildKeys[key] then return true end

	local fp = getFirePart()
	local cp = partOf(child)
	if fp and cp and (cp.Position - fp.Position).Magnitude <= 22 then
		rescuedChildren[child] = true
		if key then rescuedChildKeys[key] = true end
		return true
	end
	return false
end

local function markChildRescued(child)
	if not child then return end
	rescuedChildren[child] = true
	local key = getChildUniqueKey(child)
	if key then rescuedChildKeys[key] = true end
end

local function findItemInBag(childName)
	local itemBag = Player:FindFirstChild("ItemBag")
	if not itemBag then return nil end
	return itemBag:FindFirstChild(childName)
		or itemBag:FindFirstChild(CHILD_KEY)
		or itemBag:FindFirstChildWhichIsA("Model")
end

local function rescueSingleChild(child)
	if not child or not child.Parent then return false end
	if not (BagStoreRF and BagDropRE) then
		updateRescueStatus("❌ Thiếu remote BagStore / BagDrop")
		return false
	end

	local sack = getSackTool()
	if not sack then
		updateRescueStatus("❌ Không tìm thấy túi (Old Sack...)")
		return false
	end

	local fp = getFirePart()
	if not fp then
		updateRescueStatus("❌ Không tìm thấy Lửa trại")
		return false
	end

	local r = root()
	local cp = partOf(child)
	if not (r and cp) then return false end

	local childName = child.Name
	updateRescueStatus("🏃 Đang TP tới: " .. childName)

	r.CFrame = CFrame.new(cp.Position + Vector3.new(0, 3, 2))
	task.wait(0.25)
	if not CFG.AutoRescueChild then return false end

	updateRescueStatus("🎒 Đang cất vào balo: " .. childName)
	local bagTarget = findItemInBag(childName) or child
	local storeOk = pcall(BagStoreRF.InvokeServer, BagStoreRF, sack, bagTarget)
	if not storeOk then
		local altTarget = (bagTarget == child) and findItemInBag(childName) or child
		if altTarget then
			pcall(BagStoreRF.InvokeServer, BagStoreRF, sack, altTarget)
		end
	end
	task.wait(0.25)
	if not CFG.AutoRescueChild then return false end

	updateRescueStatus("🔥 Đang TP về lửa trại...")
	r = root()
	if r and fp and fp.Parent then
		r.CFrame = CFrame.new(fp.Position + Vector3.new(0, 4, 3))
	end
	task.wait(0.25)

	updateRescueStatus("📤 Đang thả trẻ tại lửa trại...")
	local dropTarget = (child and child.Parent) and child or findItemInBag(childName)
	if dropTarget then
		pcall(BagDropRE.FireServer, BagDropRE, sack, dropTarget, true)
	end
	task.wait(0.2)

	markChildRescued(child)
	totalChildrenSaved += 1
	updateRescueStatus(string.format("✅ Đã cứu %s (Tổng: %d)", childName, totalChildrenSaved))
	return true
end

local function processNextChildRescue()
	local r = root()
	local rPos = r and r.Position
	local nearestChild = nil
	local bestDist = math.huge

	for child, _ in pairs(childSet) do
		if child and child.Parent and not isChildAlreadyRescued(child) then
			local p = partOf(child)
			if p then
				local d = rPos and (p.Position - rPos).Magnitude or 0
				if d < bestDist then
					bestDist = d
					nearestChild = child
				end
			end
		end
	end

	if not nearestChild then
		updateRescueStatus(string.format("✅ Đã cứu hết trẻ · Tổng: %d", totalChildrenSaved))
		return
	end

	rescueSingleChild(nearestChild)
end

task.spawn(function()
	while true do
		if CFG.AutoRescueChild then
			pcall(processNextChildRescue)
			task.wait(1.0)
		else
			task.wait(0.5)
		end
	end
end)

--========================================================
-- AUTO OPEN CHEST -> LOOT AT CHEST POS + AUTO SCAN AMMO (2s)
--========================================================

local chestLootStatusText = "Chờ bật tự động mở & nhặt rương..."
local chestLootStatusLbl  = nil

local cachedChests    = {}
local openedChestDone = setmetatable({}, { __mode = "k" })
local failedChestAt   = {}
local lootedItems     = setmetatable({}, { __mode = "k" })
local failedLootAt    = {}
local claimedAmmo     = setmetatable({}, { __mode = "k" })
local failedAmmoAt    = {}

local CHEST_FAIL_COOLDOWN = 45
local LOOT_FAIL_COOLDOWN  = 35
local AMMO_FAIL_COOLDOWN  = 15
local CHEST_LOOT_RADIUS   = 12
local STEP_WAIT_SECONDS   = 2

local totalChestsOpened  = 0
local totalItemsLooted   = 0
local totalAmmoCollected = 0

local function updateChestLootStatus(txt)
	chestLootStatusText = txt
	if chestLootStatusLbl and chestLootStatusLbl.Parent then
		chestLootStatusLbl.Text = txt
	end
end

local function isChestObject(o)
	return (o:IsA("Model") or o:IsA("BasePart")) and hasKey(o.Name, CHEST_KEYS)
end

local chestCacheInitialized = false
local function initChestCache()
	if chestCacheInitialized then return end
	chestCacheInitialized = true

	local itemsFolder = Workspace:FindFirstChild("Items")
	local targetRoot  = itemsFolder or Workspace

	task.spawn(function()
		local all = targetRoot:GetDescendants()
		for i = 1, #all do
			local o = all[i]
			if isChestObject(o) then
				cachedChests[o] = true
			end
			if i % 800 == 0 then task.wait() end
		end
	end)

	targetRoot.DescendantAdded:Connect(function(o)
		if isChestObject(o) then
			cachedChests[o] = true
		end
	end)

	targetRoot.DescendantRemoving:Connect(function(o)
		cachedChests[o] = nil
		failedChestAt[o] = nil
	end)
end

local cachedAmmoList = {}
local lastAmmoScanAt = 0
local AMMO_SCAN_INTERVAL = 2.5

local function isAmmoObject(obj)
	if not obj then return false end
	local okName, name = pcall(function() return obj.Name end)
	if not okName or type(name) ~= "string" then return false end
	if hasKey(name, IGNORE_LOOT_KEYS) or hasKey(name, CHEST_KEYS) or hasKey(name, COIN_KEYS) then
		return false
	end
	return hasKey(name, AMMO_KEYS)
end

local function refreshAmmoCache(force)
	local now = os.clock()
	if not force and (now - lastAmmoScanAt) < AMMO_SCAN_INTERVAL then
		return cachedAmmoList
	end
	lastAmmoScanAt = now

	local found, seen = {}, {}
	local function tryAdd(o)
		if not o or seen[o] or claimedAmmo[o] then return end
		if isAmmoObject(o) then
			seen[o] = true
			table.insert(found, o)
		end
	end

	if getnilinstances then
		local ok, nilList = pcall(getnilinstances)
		if ok and nilList then
			for i = 1, #nilList do
				tryAdd(nilList[i])
				if i % 1000 == 0 then task.wait() end
			end
		end
	end

	local itemsFolder = Workspace:FindFirstChild("Items")
	if itemsFolder then
		for _, o in ipairs(itemsFolder:GetDescendants()) do
			tryAdd(o)
		end
	end

	cachedAmmoList = found
	return cachedAmmoList
end

local function tryClaimSingleAmmo(ammoObj)
	if not ConsumeRF or not ammoObj then return false end
	local okAlive = pcall(function() return ammoObj.Name end)
	if not okAlive then return false end

	local done, ok, res = false, false, nil
	task.spawn(function()
		ok, res = pcall(ConsumeRF.InvokeServer, ConsumeRF, ammoObj)
		done = true
	end)

	local timeout = 0
	while not done and timeout < 0.5 do
		task.wait(0.05)
		timeout += 0.05
	end

	return done and ok and res ~= false
end

local function collectScannedAmmo()
	if not ConsumeRF then return 0 end
	local list = refreshAmmoCache(false)
	if #list == 0 then return 0 end

	local now = os.clock()
	local claimed = 0
	for i = 1, math.min(#list, 12) do
		if not CFG.AutoChestAndLoot then break end
		local ammoObj = list[i]
		if claimedAmmo[ammoObj] then continue end
		local failT = failedAmmoAt[ammoObj]
		if failT and (now - failT) < AMMO_FAIL_COOLDOWN then continue end

		if tryClaimSingleAmmo(ammoObj) then
			claimedAmmo[ammoObj] = true
			claimed += 1
			totalAmmoCollected += 1
		else
			failedAmmoAt[ammoObj] = os.clock()
		end
		task.wait(0.05)
	end
	return claimed
end

local function isChestSkipped(chest, now)
	if not chest or not chest.Parent then
		cachedChests[chest] = nil
		return true
	end
	if openedChestDone[chest] then return true end
	local failT = failedChestAt[chest]
	if failT and (now - failT) < CHEST_FAIL_COOLDOWN then
		return true
	end
	return false
end

local function isLootSkipped(item, now)
	if not item or not item.Parent then return true end
	if lootedItems[item] then return true end
	local failT = failedLootAt[item]
	if failT and (now - failT) < LOOT_FAIL_COOLDOWN then
		return true
	end
	return false
end

local function isValidChestLoot(obj, targetChest)
	if not obj or not obj.Parent then return false end
	if cachedChests[obj] or obj == targetChest or obj:IsDescendantOf(targetChest) then
		return false
	end

	local ch = Player.Character
	if ch and obj:IsDescendantOf(ch) then return false end
	local inv = Player:FindFirstChild("Inventory")
	if inv and obj:IsDescendantOf(inv) then return false end
	if Players:GetPlayerFromCharacter(obj) then return false end

	local name = obj.Name
	if hasKey(name, CHEST_KEYS) or hasKey(name, COIN_KEYS) or hasKey(name, IGNORE_LOOT_KEYS) then
		return false
	end

	if obj:IsA("Tool") then return true end
	if obj:IsA("Model") then
		return (obj:FindFirstChildWhichIsA("ProximityPrompt", true)
			or obj:FindFirstChildWhichIsA("ClickDetector", true)
			or obj:FindFirstChildWhichIsA("BasePart", true)) ~= nil
	end
	return false
end

local overlapParams = OverlapParams.new()
overlapParams.FilterType = Enum.RaycastFilterType.Exclude

local function scanLootAtChestPos(chestPos, targetChest, now)
	local excludeList = { targetChest }
	if Player.Character then table.insert(excludeList, Player.Character) end
	overlapParams.FilterDescendantsInstances = excludeList

	local nearbyParts = Workspace:GetPartBoundsInRadius(chestPos, CHEST_LOOT_RADIUS, overlapParams)
	local out, seen = {}, {}

	for _, part in ipairs(nearbyParts) do
		local candidate = part:FindFirstAncestorOfClass("Tool")
			or part:FindFirstAncestorOfClass("Model")
			or part

		if candidate and not seen[candidate] then
			seen[candidate] = true
			if not isLootSkipped(candidate, now) and isValidChestLoot(candidate, targetChest) then
				local p = partOf(candidate)
				local dist = p and (p.Position - chestPos).Magnitude or CHEST_LOOT_RADIUS
				table.insert(out, { obj = candidate, dist = dist })
			end
		end
	end

	table.sort(out, function(a, b) return a.dist < b.dist end)

	local sortedItems = {}
	for i = 1, #out do
		sortedItems[i] = out[i].obj
	end
	return sortedItems
end

local function tryLootSingleItem(item)
	if not HotbarItemRF then return false end
	local inv = Player:FindFirstChild("Inventory")
	local beforeCount = inv and #inv:GetChildren() or 0

	local done, ok, res = false, false, nil
	task.spawn(function()
		ok, res = pcall(HotbarItemRF.InvokeServer, HotbarItemRF, item)
		done = true
	end)

	local timeout = 0
	while not done and timeout < 0.5 do
		task.wait(0.05)
		timeout += 0.05
	end

	if not done or not ok or res == false then
		return false
	end

	local nowInInv = (inv and item.Parent == inv) or (item.Parent == Player.Character)
	local afterCount = inv and #inv:GetChildren() or 0

	return nowInInv or afterCount > beforeCount or not item.Parent
end

local function openSingleChest(chest)
	if not OpenChestRE or not chest or not chest.Parent then return false end
	local ok = pcall(OpenChestRE.FireServer, OpenChestRE, chest)
	return ok == true
end

local function lootItemsAtChest(chest, chestPos)
	local now = os.clock()
	local items = scanLootAtChestPos(chestPos, chest, now)
	if #items == 0 then
		return 0, 0
	end

	local grabbed, skipped = 0, 0
	for i = 1, math.min(#items, 15) do
		if not CFG.AutoChestAndLoot then break end
		local item = items[i]
		if isLootSkipped(item, os.clock()) then continue end

		if tryLootSingleItem(item) then
			lootedItems[item] = true
			grabbed += 1
			totalItemsLooted += 1
		else
			failedLootAt[item] = os.clock()
			skipped += 1
		end
		task.wait(0.06)
	end
	return grabbed, skipped
end

local function processNextChestStep()
	if not OpenChestRE then
		updateChestLootStatus("❌ Thiếu remote RequestOpenItemChest")
		return
	end
	if not HotbarItemRF then
		updateChestLootStatus("❌ Thiếu remote RequestHotbarItem")
		return
	end

	initChestCache()

	local now = os.clock()
	local targetChest = nil
	local targetPos   = nil
	local r = root()
	local rPos = r and r.Position

	local bestDist = math.huge
	for chest, _ in pairs(cachedChests) do
		if not isChestSkipped(chest, now) then
			local p = partOf(chest)
			if p then
				local d = rPos and (p.Position - rPos).Magnitude or 0
				if d < bestDist then
					bestDist = d
					targetChest = chest
					targetPos = p.Position
				end
			end
		end
	end

	if not targetChest or not targetPos then
		local ammoGot = collectScannedAmmo()
		updateChestLootStatus(
			string.format("✅ Hết rương · Rương: %d | Đồ: %d | Đạn: %d (+%d)",
				totalChestsOpened, totalItemsLooted, totalAmmoCollected, ammoGot)
		)
		return
	end

	updateChestLootStatus("📦 Đang mở: " .. tostring(targetChest.Name))
	local openedOk = openSingleChest(targetChest)
	if not openedOk or not targetChest.Parent then
		failedChestAt[targetChest] = os.clock()
		local ammoGot = collectScannedAmmo()
		updateChestLootStatus(
			string.format("⏭️ Bỏ qua rương lỗi (%s) · Đạn +%d", tostring(targetChest.Name), ammoGot)
		)
		return
	end

	task.wait(0.35)
	if not CFG.AutoChestAndLoot then return end

	local currentChestPart = partOf(targetChest)
	local scanPos = currentChestPart and currentChestPart.Position or targetPos

	updateChestLootStatus("🎒 Đang nhặt đồ & đạn: " .. tostring(targetChest.Name))
	local grabbed, skipped = lootItemsAtChest(targetChest, scanPos)
	local ammoGot = collectScannedAmmo()

	if grabbed > 0 or ammoGot > 0 or targetChest.Parent then
		openedChestDone[targetChest] = true
		totalChestsOpened += 1
		updateChestLootStatus(
			string.format("📦 #%d (%s) +%d đồ +%d đạn · Tổng: %d đồ / %d đạn",
				totalChestsOpened, tostring(targetChest.Name), grabbed, ammoGot,
				totalItemsLooted, totalAmmoCollected)
		)
	else
		failedChestAt[targetChest] = os.clock()
		updateChestLootStatus(
			string.format("⏭️ Bỏ qua rương trống (%s) · Tổng: %d đồ / %d đạn",
				tostring(targetChest.Name), totalItemsLooted, totalAmmoCollected)
		)
	end
end

task.spawn(function()
	while true do
		if CFG.AutoChestAndLoot then
			pcall(processNextChestStep)
			task.wait(STEP_WAIT_SECONDS)
		else
			task.wait(0.5)
		end
	end
end)

--========================================================
-- EVENT-CACHED AUTO COLLECT ALL COINS (ZERO LAG)
--========================================================

local coinStatusText = "Chờ bật tự động nhặt xu..."
local coinStatusLbl  = nil
local totalCollectedCoins = 0
local cachedCoins = {}

local function updateCoinStatus(txt)
	coinStatusText = txt
	if coinStatusLbl and coinStatusLbl.Parent then
		coinStatusLbl.Text = txt
	end
end

local function isCoinObject(obj)
	if not obj then return false end
	local okName, name = pcall(function() return obj.Name end)
	return okName and hasKey(name, COIN_KEYS)
end

local coinCacheInitialized = false
local function initCoinCache()
	if coinCacheInitialized then return end
	coinCacheInitialized = true

	local itemsFolder = Workspace:FindFirstChild("Items")
	local targetRoot  = itemsFolder or Workspace

	task.spawn(function()
		local all = targetRoot:GetDescendants()
		for i = 1, #all do
			local o = all[i]
			if isCoinObject(o) then cachedCoins[o] = true end
			if i % 800 == 0 then task.wait() end
		end
	end)

	targetRoot.DescendantAdded:Connect(function(o)
		if isCoinObject(o) then cachedCoins[o] = true end
	end)

	targetRoot.DescendantRemoving:Connect(function(o)
		cachedCoins[o] = nil
	end)
end

local function collectAllCoinsOnce()
	if not CollectCoinRF then
		updateCoinStatus("❌ Thiếu remote RequestCollectCoints")
		return 0
	end

	initCoinCache()

	local toCollect = {}
	for coin, _ in pairs(cachedCoins) do
		if coin and coin.Parent then
			table.insert(toCollect, coin)
		else
			cachedCoins[coin] = nil
		end
	end

	if getnilinstances then
		local ok, nilList = pcall(getnilinstances)
		if ok and nilList then
			for _, obj in ipairs(nilList) do
				if isCoinObject(obj) then
					table.insert(toCollect, obj)
				end
			end
		end
	end

	if #toCollect == 0 then
		updateCoinStatus("⚠️ Chưa thấy xu (Tổng: " .. totalCollectedCoins .. ")")
		return 0
	end

	local collected = 0
	for _, coin in ipairs(toCollect) do
		if not CFG.AutoCollectCoins then break end
		local ok = pcall(CollectCoinRF.InvokeServer, CollectCoinRF, coin)
		if ok then
			cachedCoins[coin] = nil
			collected += 1
			totalCollectedCoins += 1
		end
		task.wait(0.05)
	end

	updateCoinStatus(
		string.format("🪙 Đã nhặt +%d xu (Tổng: " .. totalCollectedCoins .. ")", collected)
	)
	return collected
end

task.spawn(function()
	while true do
		task.wait(1.2)
		if CFG.AutoCollectCoins then
			pcall(collectAllCoinsOnce)
		end
	end
end)

--========================================================
-- COUNTER, KILL AURA & TP AUTO CHOP (EXACT TREE NAMES + ANTI-LOSE AXE FIX)
--========================================================

local hitCounter = 0
local lastEquippedChopTool = nil
local isChoppingNow = false

local function resetCounter()
	hitCounter = 0
	lastEquippedChopTool = nil
	isChoppingNow = false
end

Workspace.ChildAdded:Connect(function(c)
	if c.Name == "Map" then
		resetCounter()
		cachedFirePart = nil
	end
end)
Workspace.ChildRemoved:Connect(function(c)
	if c.Name == "Map" then
		resetCounter()
		cachedFirePart = nil
	end
end)

local function nextId()
	hitCounter += 1
	return hitCounter .. "_" .. TOOL_SUFFIX
end

local function fireWoodChopSound()
	local head = getHeadPart()
	if ReplicateSoundRE and head then
		pcall(ReplicateSoundRE.FireServer, ReplicateSoundRE, "FireAllClients", "WoodChop", {
			Instance = head,
			Volume = 0.4,
		})
	end
end

local function isToolValid(tool)
	if not tool or not tool.Parent then return false end
	local inv = Player:FindFirstChild("Inventory")
	local char = Player.Character
	return (inv and tool:IsDescendantOf(inv)) or (char and tool:IsDescendantOf(char))
end

local function isToolEquippedOnChar(tool)
	local char = Player.Character
	return char and tool and tool.Parent == char
end

local function hitOnce(tool, target, p)
	if not DamageRF or not target or not target.Parent or not p or not p.Parent then
		return false
	end
	if not isToolValid(tool) then return false end
	local ok = pcall(DamageRF.InvokeServer, DamageRF,
		target, tool, nextId(), CFrame.new(p.Position), false)
	if HitSoundRE and ok then
		pcall(HitSoundRE.FireServer, HitSoundRE, "FireAllClients", target, tool)
	end
	return ok == true
end

local function equipToolSafe(tool)
	if not (EquipRE and isToolValid(tool)) then return end
	if isToolEquippedOnChar(tool) then return end
	pcall(EquipRE.FireServer, EquipRE, "FireAllClients", tool)
	task.wait(0.06)
end

local function unequipToolSafe(tool)
	-- Chỉ unequip nếu tool vẫn tồn tại hợp lệ và đang thực sự cầm trên Character
	if not (UnequipRE and isToolValid(tool)) then return end
	if not isToolEquippedOnChar(tool) then return end
	pcall(UnequipRE.FireServer, UnequipRE, "FireAllClients", tool)
	if lastEquippedChopTool == tool then
		lastEquippedChopTool = nil
	end
	task.wait(0.05)
end

-- Chỉ gửi lệnh Equip nếu rìu chưa được cầm trên Character (tránh spam Equip gây mất rìu)
local function equipChopToolOnce(tool)
	if not (EquipRE and isToolValid(tool)) then return false end
	if isToolEquippedOnChar(tool) then
		lastEquippedChopTool = tool
		return true
	end
	if lastEquippedChopTool ~= tool or not isToolEquippedOnChar(tool) then
		pcall(EquipRE.FireServer, EquipRE, "FireAllClients", tool)
		lastEquippedChopTool = tool
		task.wait(0.08)
	end
	return isToolValid(tool)
end

local function swingBatch(tool, targets)
	if not (EquipRE and UnequipRE and DamageRF) or #targets == 0 then return end
	if not isToolValid(tool) then return end

	-- Nếu AutoChop đang giữ rìu thì không để KillAura tự ý unequip làm mất đồng bộ
	local sharedWithChop = isChoppingNow and (lastEquippedChopTool == tool)

	equipToolSafe(tool)
	local limit = math.min(#targets, MAX_KILL_PER_CYCLE)
	for i = 1, limit do
		if not CFG.KillAura then break end
		local t = targets[i]
		if t.m and t.m.Parent and t.p and t.p.Parent then
			local h = t.m:FindFirstChildWhichIsA("Humanoid", true)
			if h and h.Health > 0 then
				hitOnce(tool, t.m, t.p)
			end
		end
		if i < limit then task.wait(0.05) end
	end

	if not sharedWithChop and not CFG.AutoChop then
		unequipToolSafe(tool)
	end
end

local SCAN_FOLDERS = { "Characters","Enemies","Mobs","NPCs" }
local function getAnimals(r)
	local out, ch, seen = {}, Player.Character, {}
	local maxDSq = CFG.KillRadius * CFG.KillRadius
	local rPos   = r.Position

	local function tryAdd(o)
		if seen[o] or o == ch or childSet[o] then return end
		seen[o] = true
		if Players:GetPlayerFromCharacter(o) then return end
		if string.find(o.Name, CHILD_KEY, 1, true) then return end
		local h = o:FindFirstChildWhichIsA("Humanoid", true)
		if not (h and h.Health > 0) then return end
		local p = partOf(o); if not p then return end
		local d = p.Position - rPos
		local dSq = d.X*d.X + d.Y*d.Y + d.Z*d.Z
		if dSq <= maxDSq then table.insert(out, {m=o, p=p, dSq=dSq}) end
	end

	for _, o in ipairs(Workspace:GetChildren()) do
		if o:IsA("Model") then tryAdd(o) end
	end
	for _, fn in ipairs(SCAN_FOLDERS) do
		local folder = Workspace:FindFirstChild(fn)
		if folder then
			for _, o in ipairs(folder:GetChildren()) do
				if o:IsA("Model") then tryAdd(o) end
			end
		end
	end
	table.sort(out, function(a,b) return a.dSq < b.dSq end)
	return out
end

local skipTree = {}

local function findNilEquivalent(targetName, targetDebugId)
	if not (targetName and targetDebugId and getnilinstances) then return nil end
	local okList, nilList = pcall(getnilinstances)
	if not okList or not nilList then return nil end

	for _, inst in ipairs(nilList) do
		local okN, n = pcall(function() return inst.Name end)
		local okD, d = pcall(function() return inst:GetDebugId() end)
		if okN and okD and n == targetName and d == targetDebugId then
			return inst
		end
	end
	return nil
end

-- Kiểm tra xem cây đã gãy hoặc bị phá hủy chưa
local function isTreeBroken(treeModel, treePart)
	if not treeModel or not treeModel.Parent then return true end
	if not treePart or not treePart.Parent then return true end

	local hpAttr = treeModel:GetAttribute("Health") or treeModel:GetAttribute("HP")
	if type(hpAttr) == "number" and hpAttr <= 0 then return true end

	local hpVal = treeModel:FindFirstChild("Health", true) or treeModel:FindFirstChild("HP", true)
	if hpVal and hpVal:IsA("ValueBase") and type(hpVal.Value) == "number" and hpVal.Value <= 0 then
		return true
	end

	local h = treeModel:FindFirstChildWhichIsA("Humanoid", true)
	if h and h.Health <= 0 then return true end

	if treePart:IsA("BasePart") and not treePart.Anchored then
		return true
	end

	return false
end

-- Chuẩn hóa chuỗi để so khớp linh hoạt (bỏ khoảng trắng/chữ hoa thường)
local function normalizeName(str)
	if type(str) ~= "string" then return "" end
	return string.lower(string.gsub(str, "%s+", ""))
end

local function isSmallTreeName(name)
	local n = normalizeName(name)
	return n == "smalltree" or string.find(n, "smalltree", 1, true) ~= nil
end

local function isBigTreeName(name)
	local n = normalizeName(name)
	return n == "treebig1" or n == "treebig2"
		or string.find(n, "treebig1", 1, true) ~= nil
		or string.find(n, "treebig2", 1, true) ~= nil
end

-- Chỉ Strong Axe, Chainsaw, Admin Axe mới chặt được treebig1 và treebig2
local function canChopBigTree(tool)
	if not tool then return false end
	return HEAVY_AXES[tool.Name] == true
end

-- Trả về:
--   "normal" -> Small Tree (mọi rìu chặt được)
--   "big"    -> treebig1, treebig 2 (chỉ Strong Axe, Chainsaw, Admin Axe)
--   nil      -> không phải loại cây hợp lệ hoặc đã gãy
local function classifyTree(treeModel)
	if not treeModel or not treeModel.Parent or not treeModel:IsA("Model") then
		return nil
	end

	local p = partOf(treeModel)
	if not p or isTreeBroken(treeModel, p) then
		return nil
	end

	if isSmallTreeName(treeModel.Name) then
		return "normal"
	end

	if isBigTreeName(treeModel.Name) then
		return "big"
	end

	return nil
end

local function getTrees(r, maxRadius, tool)
	local out = {}
	local useLimit = type(maxRadius) == "number" and maxRadius > 0
	local maxDSq = useLimit and (maxRadius * maxRadius) or math.huge
	local rPos   = r.Position
	local now    = os.clock()
	local allowBig = canChopBigTree(tool)

	local function tryAddTree(t)
		if skipTree[t] and now < skipTree[t] then return end

		local treeType = classifyTree(t)
		if not treeType then
			return
		end
		if treeType == "big" and not allowBig then
			return -- Không có Strong Axe / Chainsaw / Admin Axe thì bỏ qua treebig1 & treebig 2
		end

		local p = partOf(t); if not p then return end
		local d = p.Position - rPos
		local dSq = d.X*d.X + d.Y*d.Y + d.Z*d.Z
		if dSq <= maxDSq then
			table.insert(out, { m = t, p = p, dSq = dSq, treeType = treeType })
		end
	end

	local map = Workspace:FindFirstChild("Map")
	local folders = {
		map and map:FindFirstChild("Foliage"),
		map and map:FindFirstChild("Trees"),
		Workspace:FindFirstChild("Foliage"),
		Workspace:FindFirstChild("Trees"),
	}

	for _, folder in ipairs(folders) do
		if folder then
			for _, t in ipairs(folder:GetChildren()) do
				tryAddTree(t)
			end
		end
	end

	if #out == 0 then
		local scanRoot = map or Workspace
		for _, o in ipairs(scanRoot:GetDescendants()) do
			if o:IsA("Model") then
				tryAddTree(o)
			end
		end
	end

	table.sort(out, function(a, b) return a.dSq < b.dSq end)
	return out
end

local function spamChopDamageOnce(tool, treeModel, treePart, savedName, savedDebugId, fallbackPos)
	if not DamageRF or not isToolValid(tool) then return false end

	local actualTarget = (treeModel and treeModel.Parent)
		and treeModel
		or findNilEquivalent(savedName, savedDebugId)
	if not actualTarget then return false end

	local hitPos = (treePart and treePart.Parent) and treePart.Position or fallbackPos
	local rr = root()
	local hitCF
	if rr and hitPos then
		local lookPos = Vector3.new(hitPos.X, rr.Position.Y, hitPos.Z)
		hitCF = CFrame.lookAt(hitPos, lookPos)
	elseif hitPos then
		hitCF = CFrame.new(hitPos)
	else
		hitCF = CFrame.new(0, 0, 0)
	end

	fireWoodChopSound()

	local ok = pcall(DamageRF.InvokeServer, DamageRF,
		actualTarget, tool, nextId(), hitCF, false)

	if HitSoundRE and ok then
		pcall(HitSoundRE.FireServer, HitSoundRE, "FireAllClients", actualTarget, tool)
	end

	return ok == true
end

local function tpToTreeAndChop(treeModel, treePart, initialTool)
	local r = root()
	if not (r and treeModel and treeModel.Parent and treePart and treePart.Parent) then
		return false
	end
	if not isToolValid(initialTool) then
		return false
	end

	local savedName = treeModel.Name
	local okDbg, savedDebugId = pcall(function() return treeModel:GetDebugId() end)
	if not okDbg then savedDebugId = nil end

	local targetPos = treePart.Position
	r.CFrame = CFrame.lookAt(targetPos + Vector3.new(0, 2.5, -3), targetPos)
	task.wait(0.08)

	isChoppingNow = true
	local lockedTool = initialTool
	if not equipChopToolOnce(lockedTool) then
		isChoppingNow = false
		return false
	end

	local swings = 0
	local maxSwings = 45
	local failedHits = 0
	local spamWait = math.min(math.max(CFG.Delay or 0.06, 0.05), 0.12)

	-- Spam ToolDamageObject liên tục cho đến khi cây gãy hẳn mới chuyển cây khác
	while CFG.AutoChop and not isTreeBroken(treeModel, treePart) and swings < maxSwings do
		local rr = root()
		if not rr then break end

		-- Kiểm tra rìu đang khóa còn hợp lệ không; chỉ tìm rìu khác nếu rìu cũ không còn
		if not isToolValid(lockedTool) then
			local fallback = getChopTool()
			if not isToolValid(fallback) then break end
			lockedTool = fallback
			equipChopToolOnce(lockedTool)
		elseif not isToolEquippedOnChar(lockedTool) then
			equipChopToolOnce(lockedTool)
		end

		local curTreePos = (treePart and treePart.Parent) and treePart.Position or targetPos
		if (rr.Position - curTreePos).Magnitude > 7 then
			rr.CFrame = CFrame.lookAt(curTreePos + Vector3.new(0, 2.5, -3), curTreePos)
		end

		local hitOk = spamChopDamageOnce(lockedTool, treeModel, treePart, savedName, savedDebugId, curTreePos)
		if hitOk then
			failedHits = 0
		else
			failedHits += 1
			if failedHits >= 8 then
				break
			end
		end

		swings += 1
		task.wait(spamWait)
	end

	isChoppingNow = false
	return isTreeBroken(treeModel, treePart)
end

-- Luồng riêng cho Kill Aura
task.spawn(function()
	while true do
		if CFG.KillAura then
			local r, tool = root(), getKillTool()
			if r and isToolValid(tool) then
				local targets = getAnimals(r)
				if #targets > 0 then
					pcall(swingBatch, tool, targets)
				end
			end
			task.wait(CFG.Delay)
		else
			task.wait(0.25)
		end
	end
end)

-- Luồng riêng cho TP Auto Chop
task.spawn(function()
	while true do
		if CFG.AutoChop then
			local r, tool = root(), getChopTool()
			if r and isToolValid(tool) then
				local trees = getTrees(r, CFG.ChopRadius, tool)
				if #trees == 0 then
					trees = getTrees(r, nil, tool)
				end
				if #trees > 0 then
					local chosen = trees[1]
					local broken = false
					pcall(function()
						broken = tpToTreeAndChop(chosen.m, chosen.p, tool)
					end)
					isChoppingNow = false
					if broken then
						skipTree[chosen.m] = os.clock() + 60
					else
						skipTree[chosen.m] = os.clock() + 5
					end
				end
			end
			task.wait(0.12)
		else
			-- Không tự động gọi unequipToolSafe khi tắt AutoChop để tránh server xóa/desync rìu
			isChoppingNow = false
			lastEquippedChopTool = nil
			task.wait(0.25)
		end
	end
end)

--========================================================
-- MOVEMENT, NOCLIP & FLY
--========================================================

local noclipBackup = {}
local function restoreNoclip()
	for part, wasCollide in pairs(noclipBackup) do
		if part and part.Parent and part:IsA("BasePart") then
			pcall(function() part.CanCollide = wasCollide end)
		end
	end
	noclipBackup = {}
end

RunService.Heartbeat:Connect(function()
	local h = hum(); if not h then return end
	if CFG.Speed then h.WalkSpeed = CFG.WalkSpeed end
	if CFG.Jump  then h.UseJumpPower = true; h.JumpPower = CFG.JumpPower end
end)

RunService.Stepped:Connect(function()
	local c = Player.Character; if not c then return end
	if CFG.Noclip then
		for _, obj in ipairs(c:GetDescendants()) do
			if obj:IsA("BasePart") then
				if noclipBackup[obj] == nil then noclipBackup[obj] = obj.CanCollide end
				if obj.CanCollide then obj.CanCollide = false end
			end
		end
	elseif next(noclipBackup) then
		restoreNoclip()
	end
end)

local flyBV, flyBG, flyConn
local padUp, padDown = false, false

local function stopFly()
	if flyConn then flyConn:Disconnect(); flyConn = nil end
	if flyBV and flyBV.Parent then flyBV:Destroy() end
	if flyBG and flyBG.Parent then flyBG:Destroy() end
	flyBV, flyBG = nil, nil
	local h = hum(); if h then h.PlatformStand = false end
end

local function startFly()
	stopFly()
	local r, h = root(), hum(); if not (r and h) then return end
	h.PlatformStand = true
	flyBV = Instance.new("BodyVelocity")
	flyBV.MaxForce = Vector3.new(1e6,1e6,1e6)
	flyBV.Velocity = Vector3.zero; flyBV.Parent = r
	flyBG = Instance.new("BodyGyro")
	flyBG.MaxTorque = Vector3.new(1e6,1e6,1e6)
	flyBG.P = 1e4; flyBG.Parent = r
	flyConn = RunService.Heartbeat:Connect(function()
		local cam = Workspace.CurrentCamera
		local rr, hh = root(), hum()
		if not (cam and rr and hh and flyBV and flyBV.Parent) then return end
		local md = hh.MoveDirection
		local camLook  = cam.CFrame.LookVector
		local camRight = cam.CFrame.RightVector
		local flatLook  = Vector3.new(camLook.X,  0, camLook.Z)
		local flatRight = Vector3.new(camRight.X, 0, camRight.Z)
		if flatLook.Magnitude  > 0 then flatLook  = flatLook.Unit  end
		if flatRight.Magnitude > 0 then flatRight = flatRight.Unit end
		local moveDir = flatLook * (-md.Z) + flatRight * md.X
		local vy = 0
		if UIS:IsKeyDown(Enum.KeyCode.Space)       or padUp   then vy =  1 end
		if UIS:IsKeyDown(Enum.KeyCode.LeftControl) or padDown then vy = -1 end
		flyBV.Velocity = moveDir * CFG.FlySpeed + Vector3.new(0, vy * CFG.FlySpeed, 0)
		flyBG.CFrame   = CFrame.new(rr.Position, rr.Position + flatLook)
	end)
end

Player.CharacterAdded:Connect(function()
	task.wait(0.5); resetCounter()
	if CFG.Fly then startFly() end
end)

--========================================================
-- FULLBRIGHT & FIX LAG PRO
--========================================================

local fbPrev, fbConn
local function applyFB()
	Lighting.Brightness = 3; Lighting.ClockTime = 12; Lighting.FogEnd = 1e6
	Lighting.Ambient = Color3.fromRGB(180,180,180)
	Lighting.OutdoorAmbient = Color3.fromRGB(180,180,180)
end
local function setFullbright(on)
	if on then
		if not fbPrev then fbPrev = {
			B=Lighting.Brightness, C=Lighting.ClockTime,
			F=Lighting.FogEnd, A=Lighting.Ambient, OA=Lighting.OutdoorAmbient
		} end
		applyFB()
		if not fbConn then
			fbConn = RunService.Heartbeat:Connect(function()
				if CFG.Fullbright then applyFB() end
			end)
		end
	else
		if fbConn then fbConn:Disconnect(); fbConn = nil end
		if fbPrev then
			Lighting.Brightness = fbPrev.B; Lighting.ClockTime = fbPrev.C
			Lighting.FogEnd = fbPrev.F; Lighting.Ambient = fbPrev.A
			Lighting.OutdoorAmbient = fbPrev.OA; fbPrev = nil
		end
	end
end

local LagBackup = {}
local lagConn, lagGen = nil, 0
local HEAVY_MATS = {
	[Enum.Material.Neon]          = Enum.Material.SmoothPlastic,
	[Enum.Material.Glass]         = Enum.Material.SmoothPlastic,
	[Enum.Material.ForceField]    = Enum.Material.SmoothPlastic,
	[Enum.Material.DiamondPlate]  = Enum.Material.SmoothPlastic,
	[Enum.Material.CorrodedMetal] = Enum.Material.SmoothPlastic,
	[Enum.Material.Foil]          = Enum.Material.SmoothPlastic,
	[Enum.Material.WoodPlanks]    = Enum.Material.SmoothPlastic,
	[Enum.Material.Cobblestone]   = Enum.Material.SmoothPlastic,
}
local function saveProp(obj, prop)
	local t = LagBackup[obj]
	if not t then t = {}; LagBackup[obj] = t end
	if t[prop] == nil then
		local ok, val = pcall(function() return obj[prop] end)
		if ok then t[prop] = val end
	end
end
local function optimizeObj(obj)
	if not CFG.FixLag or not obj or not obj.Parent then return end
	if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam")
	or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles")
	or obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
		saveProp(obj, "Enabled"); pcall(function() obj.Enabled = false end); return
	end
	if obj:IsA("BasePart") then
		saveProp(obj, "CastShadow"); pcall(function() obj.CastShadow = false end)
		saveProp(obj, "Material")
		local mat = HEAVY_MATS[obj.Material] or Enum.Material.Plastic
		if obj.Material ~= mat then pcall(function() obj.Material = mat end) end
	end
end
local function setFixLag(v)
	if v then
		if CFG.FixLag then return end; CFG.FixLag = true; lagGen += 1; local gen = lagGen
		for _, prop in ipairs({"GlobalShadows","EnvironmentDiffuseScale","EnvironmentSpecularScale","ShadowSoftness"}) do
			saveProp(Lighting, prop)
		end
		pcall(function()
			Lighting.GlobalShadows = false; Lighting.EnvironmentDiffuseScale = 0
			Lighting.EnvironmentSpecularScale = 0; Lighting.ShadowSoftness = 0
		end)
		if not lagConn then
			lagConn = Workspace.DescendantAdded:Connect(function(obj)
				if CFG.FixLag then task.defer(optimizeObj, obj) end
			end)
		end
		task.spawn(function()
			local all = Workspace:GetDescendants()
			for i = 1, #all do
				if not CFG.FixLag or gen ~= lagGen then return end
				optimizeObj(all[i])
				if i % 500 == 0 then task.wait() end
			end
		end)
	else
		if not CFG.FixLag then return end; CFG.FixLag = false; lagGen += 1
		if lagConn then lagConn:Disconnect(); lagConn = nil end
		for obj, props in pairs(LagBackup) do
			if obj and obj.Parent then
				for prop, val in pairs(props) do pcall(function() obj[prop] = val end) end
			end
		end
		LagBackup = {}
	end
end

--========================================================
-- AUTO EAT CARROT & ESP TRẺ (EVENT-DRIVEN)
--========================================================

local eatStatusText = "Chờ bật ăn carrot..."
local eatStatusLbl  = nil

local function updateEatStatus(txt)
	eatStatusText = txt
	if eatStatusLbl and eatStatusLbl.Parent then
		eatStatusLbl.Text = txt
	end
end

task.spawn(function()
	while true do
		task.wait(2)
		if CFG.AutoEat then
			if not ConsumeRF then
				updateEatStatus("❌ Thiếu remote RequestConsumeItem")
				continue
			end
			if not getnilinstances then
				updateEatStatus("❌ Executor thiếu getnilinstances")
				continue
			end
			local carrots = getNilByName("Carrot")
			if #carrots == 0 then
				updateEatStatus("⚠️ Không tìm thấy Carrot")
				continue
			end
			local ate = 0
			for _, carrot in ipairs(carrots) do
				if not CFG.AutoEat then break end
				local ok = pcall(ConsumeRF.InvokeServer, ConsumeRF, carrot)
				if ok then
					ate += 1
					updateEatStatus(string.format("✅ Đã ăn %d carrot", ate))
				end
				task.wait(0.3)
			end
		end
	end
end)

local espTag = {}
local function clearESPChild()
	for _, pair in pairs(espTag) do
		if pair.hl  and pair.hl.Parent  then pair.hl:Destroy()  end
		if pair.bb  and pair.bb.Parent  then pair.bb:Destroy()  end
		if pair.conn then pair.conn:Disconnect() end
	end
	espTag = {}
end

task.spawn(function()
	while true do
		task.wait(0.6)
		if CFG.ESPChild then
			for m, pair in pairs(espTag) do
				if not m.Parent then
					if pair.hl  and pair.hl.Parent  then pair.hl:Destroy()  end
					if pair.bb  and pair.bb.Parent  then pair.bb:Destroy()  end
					if pair.conn then pair.conn:Disconnect() end
					espTag[m] = nil
				end
			end
			for m, _ in pairs(childSet) do
				if m.Parent and not espTag[m] then
					local saved = isChildAlreadyRescued(m)
					local espColor = saved and Color3.fromRGB(100, 255, 130) or Color3.fromRGB(255, 220, 80)

					local hl = Instance.new("Highlight")
					hl.FillColor        = espColor
					hl.OutlineColor     = espColor
					hl.FillTransparency = 0.45
					hl.Adornee = m; hl.Parent = m

					local adornPart = partOf(m)
					local bb, lblConn
					if adornPart then
						bb = Instance.new("BillboardGui")
						bb.Adornee     = adornPart
						bb.Size        = UDim2.new(0,170,0,46)
						bb.StudsOffset = Vector3.new(0,3,0)
						bb.AlwaysOnTop = true; bb.Parent = adornPart

						local lbl = Instance.new("TextLabel")
						lbl.Size               = UDim2.new(1,0,1,0)
						lbl.BackgroundTransparency = 1
						lbl.TextColor3         = espColor
						lbl.TextStrokeColor3   = Color3.new(0,0,0)
						lbl.TextStrokeTransparency = 0.3
						lbl.Font               = Enum.Font.GothamBold
						lbl.TextSize           = 13; lbl.Parent = bb

						lblConn = RunService.Heartbeat:Connect(function()
							if not m.Parent or not CFG.ESPChild then
								lblConn:Disconnect(); return
							end
							local rr = root(); local pp = partOf(m)
							local statusTag = isChildAlreadyRescued(m) and " [Đã cứu]" or ""
							if rr and pp then
								local dist = math.floor((pp.Position - rr.Position).Magnitude)
								lbl.Text = m.Name .. statusTag .. "\n📍 " .. dist .. " studs"
							else
								lbl.Text = m.Name .. statusTag
							end
						end)
					end
					espTag[m] = { hl=hl, bb=bb, conn=lblConn }
				end
			end
		else
			if next(espTag) then clearESPChild() end
		end
	end
end)

--========================================================
-- TP, PULL, FUEL & PLANT ACTIONS (OPTIMIZED)
--========================================================

local function tpTo(m)
	local r, p = root(), partOf(m)
	if r and p then r.CFrame = CFrame.new(p.Position + Vector3.new(0,4,0)) end
end

local function tpCampfire()
	local fp = getFirePart(); local r = root()
	if fp and r then r.CFrame = CFrame.new(fp.Position + Vector3.new(0,4,0)) end
end

local function pullPlayerToMe(plr)
	if not plr or plr == Player then return false end
	local r = root()
	local targetRoot = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
	if r and targetRoot then
		targetRoot.CFrame = r.CFrame * CFrame.new(0,1,-4)
		return true
	end
	return false
end

local function pullAllExact(nameList)
	local r = root(); if not r then return 0 end
	local ch = Player.Character
	local found = {}
	local itemsFolder = Workspace:FindFirstChild("Items")
	local searchList  = itemsFolder and itemsFolder:GetDescendants() or Workspace:GetChildren()

	for _, o in ipairs(searchList) do
		if ch and o:IsDescendantOf(ch) then continue end
		if Players:GetPlayerFromCharacter(o) then continue end
		if (o:IsA("Model") or o:IsA("BasePart")) and matchesExact(o.Name, nameList) then
			local p = partOf(o)
			if p then
				local d = (p.Position - r.Position).Magnitude
				table.insert(found, { obj=o, dist=d })
			end
		end
	end
	table.sort(found, function(a,b) return a.dist < b.dist end)
	local count = 0
	for i = 1, math.min(#found, 20) do
		if not (DragRE and StopDragRE) then break end
		local o  = found[i].obj
		local rr = root(); if not rr then break end
		local spreadX = ((count % 3) - 1) * 1.4
		local liftY   = 1.2 + math.floor(count / 3) * 0.6
		local goalCF  = rr.CFrame * CFrame.new(spreadX, liftY, -3)
		if o:IsA("Model") then
			if not o.PrimaryPart then
				pcall(function() o.PrimaryPart = partOf(o) end)
			end
			pcall(DragRE.FireServer, DragRE, o)
			if o.PrimaryPart then
				pcall(function() o:SetPrimaryPartCFrame(goalCF) end)
			else
				pcall(function() o:PivotTo(goalCF) end)
			end
		elseif o:IsA("BasePart") then
			pcall(DragRE.FireServer, DragRE, o)
			pcall(function() o.CFrame = goalCF end)
		end
		task.wait(0.04)
		pcall(StopDragRE.FireServer, StopDragRE, o)
		count += 1; task.wait(0.08)
	end
	return count
end

local droppedAt = {}
local function dumpFuelToFire()
	local fp = getFirePart()
	if not (fp and DragRE and StopDragRE) then return 0 end
	local ch, count, seen = Player.Character, 0, {}
	local itemsFolder = Workspace:FindFirstChild("Items")
	local searchList  = itemsFolder and itemsFolder:GetDescendants() or Workspace:GetChildren()

	for _, o in ipairs(searchList) do
		if not o:IsA("Model") then continue end
		if seen[o] then continue end
		if ch and o:IsDescendantOf(ch) then continue end
		if Players:GetPlayerFromCharacter(o) then continue end
		if (droppedAt[o] or 0) + 20 > os.clock() then continue end
		if not matchesExact(o.Name, FUEL_EXACT) then continue end
		seen[o] = true
		if not o.PrimaryPart then pcall(function() o.PrimaryPart = partOf(o) end) end
		if not o.PrimaryPart then continue end
		droppedAt[o] = os.clock()
		local offset = Vector3.new(math.random(-20,20)/10, 7+count*0.4, math.random(-20,20)/10)
		pcall(DragRE.FireServer, DragRE, o)
		pcall(function() o:SetPrimaryPartCFrame(CFrame.new(fp.Position + offset)) end)
		task.wait(0.05)
		pcall(StopDragRE.FireServer, StopDragRE, o)
		count += 1; task.wait(0.1)
	end
	return count
end

task.spawn(function()
	while true do
		task.wait(5)
		if CFG.AutoDrop then pcall(dumpFuelToFire) end
	end
end)

local function plantAllSaplings(statusLbl)
	local function setStatus(txt)
		if statusLbl and statusLbl.Parent then
			statusLbl.Text = txt
		end
	end
	if not PlantRE then
		setStatus("❌ Thiếu remote RequestPlantItem"); return 0
	end
	if not getnilinstances then
		setStatus("❌ Executor thiếu getnilinstances"); return 0
	end
	setStatus("🔍 Đang quét mầm cây...")
	local saplings = getNilByName("Sapling")
	if #saplings == 0 then
		setStatus("⚠️ Không tìm thấy mầm cây nào"); return 0
	end
	setStatus(string.format("🌱 Tìm thấy %d mầm — đang trồng...", #saplings))
	local r = root()
	if not r then
		setStatus("❌ Không tìm thấy nhân vật"); return 0
	end
	local basePos = r.CFrame.Position - Vector3.new(0, 3, 0)
	local count = 0
	for _, sapling in ipairs(saplings) do
		local ok0 = pcall(function() return sapling.Name end)
		if not ok0 then continue end
		local offset = Vector3.new(math.random(-3,3)/10, 0, math.random(-3,3)/10)
		local ok2 = pcall(PlantRE.InvokeServer, PlantRE, sapling, basePos + offset)
		if StopDragRE then pcall(StopDragRE.FireServer, StopDragRE, sapling) end
		if ok2 then count += 1 end
		task.wait(0.15)
	end
	if count > 0 then
		setStatus(string.format("✅ Đã trồng %d/%d mầm cây", count, #saplings))
	else
		setStatus("⚠️ Trồng thất bại — thử đứng sát đất")
	end
	return count
end

local function revealMap()
	if not MapRE then return end
	local cells = {}
	for row = 1, 64 do
		for col = 1, 64 do table.insert(cells, {Row=row, Col=col}) end
	end
	for i = 1, #cells, 200 do
		local b = {}
		for j = i, math.min(i+199, #cells) do table.insert(b, cells[j]) end
		pcall(MapRE.FireServer, MapRE, b, {}); task.wait(0.1)
	end
end

--========================================================
-- SUNSET UI LIBRARY
--========================================================

local Library = {}

function Library:CreateWindow(title)
	local gui = Instance.new("ScreenGui")
	gui.Name = "SunsetUI"
	gui.Parent = CoreGui.Parent and CoreGui or PlayerGui
	pcall(function() gui.Parent = CoreGui end)
	if gui.Parent ~= CoreGui then
		gui.Parent = PlayerGui
	end
	gui.IgnoreGuiInset = true
	gui.ResetOnSpawn = false
	gui.DisplayOrder = 100

	local currentColor = Color3.fromRGB(255, 128, 0)
	local toggleButton = nil
	local toggleButtonSize = 40
	local mainFrame = nil
	local sidebarFrame = nil
	local titleLabel = nil
	local contentFrame = nil

	local function updateThemeColor(newColor)
		currentColor = newColor
		if mainFrame and mainFrame:FindFirstChild("UIStroke") then
			mainFrame.UIStroke.Color = currentColor
		end
		if sidebarFrame and sidebarFrame:FindFirstChild("UIStroke") then
			sidebarFrame.UIStroke.Color = currentColor
		end
		if titleLabel then
			titleLabel.TextColor3 = currentColor
		end
		if toggleButton then
			toggleButton.BackgroundColor3 = currentColor
		end
		if contentFrame then
			for _, tab in ipairs(contentFrame:GetChildren()) do
				if tab:IsA("ScrollingFrame") then
					tab.ScrollBarImageColor3 = currentColor
				end
			end
		end
	end

	local function updateToggleButtonSize(newSize)
		toggleButtonSize = newSize
		if toggleButton then
			toggleButton.Size = UDim2.new(0, newSize, 0, newSize)
			toggleButton.Position = UDim2.new(0, 10, 0.5, -(newSize / 2))
		end
	end

	local function Notify(titulo, texto, duracao)
		pcall(function()
			game:GetService("StarterGui"):SetCore("SendNotification", {
				Title = titulo,
				Text = texto,
				Duration = duracao or 5,
			})
		end)
	end

	toggleButton = Instance.new("ImageButton")
	toggleButton.Size = UDim2.new(0, toggleButtonSize, 0, toggleButtonSize)
	toggleButton.Position = UDim2.new(0, 10, 0.5, -(toggleButtonSize / 2))
	toggleButton.BackgroundColor3 = currentColor
	toggleButton.Image = "rbxassetid://97939372251102"
	toggleButton.ZIndex = 20
	toggleButton.Parent = gui
	Instance.new("UICorner", toggleButton).CornerRadius = UDim.new(1, 0)

	local draggingBtn, dragStartBtn, startPosBtn, movedBtn = false, nil, nil, false
	toggleButton.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			draggingBtn = true
			movedBtn = false
			dragStartBtn = input.Position
			startPosBtn = toggleButton.Position
		end
	end)
	toggleButton.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			draggingBtn = false
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if draggingBtn and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
			local delta = input.Position - dragStartBtn
			if delta.Magnitude > 5 then movedBtn = true end
			toggleButton.Position = UDim2.new(startPosBtn.X.Scale, startPosBtn.X.Offset + delta.X, startPosBtn.Y.Scale, startPosBtn.Y.Offset + delta.Y)
		end
	end)

	mainFrame = Instance.new("Frame")
	mainFrame.Size = UDim2.new(0, 450, 0, 380)
	mainFrame.Position = UDim2.new(0.5, -225, 0.5, -190)
	mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	mainFrame.BackgroundTransparency = 0.2
	mainFrame.Parent = gui
	Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)

	local draggingMain, dragStart, startPos = false, nil, nil
	mainFrame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			draggingMain = true
			dragStart = input.Position
			startPos = mainFrame.Position
		end
	end)
	mainFrame.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
			draggingMain = false
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if draggingMain and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
			local delta = input.Position - dragStart
			mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)

	local strokeMain = Instance.new("UIStroke")
	strokeMain.Color = currentColor
	strokeMain.Thickness = 1
	strokeMain.Parent = mainFrame

	sidebarFrame = Instance.new("Frame")
	sidebarFrame.Size = UDim2.new(0, 90, 1, 0)
	sidebarFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	sidebarFrame.BackgroundTransparency = 0.2
	sidebarFrame.Parent = mainFrame
	Instance.new("UICorner", sidebarFrame).CornerRadius = UDim.new(0, 8)

	local strokeSidebar = Instance.new("UIStroke")
	strokeSidebar.Color = currentColor
	strokeSidebar.Parent = sidebarFrame

	titleLabel = Instance.new("TextLabel")
	titleLabel.Size = UDim2.new(1, 0, 0, 30)
	titleLabel.Text = title
	titleLabel.TextColor3 = currentColor
	titleLabel.BackgroundTransparency = 1
	titleLabel.TextSize = 14
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.Parent = sidebarFrame

	contentFrame = Instance.new("Frame")
	contentFrame.Size = UDim2.new(1, -90, 1, 0)
	contentFrame.Position = UDim2.new(0, 90, 0, 0)
	contentFrame.BackgroundTransparency = 1
	contentFrame.Parent = mainFrame

	local currentTab
	local tabY = 35
	local tabOrderList = {}

	local windowFunctions = {}

	function windowFunctions:CreateTab(name)
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(1, -10, 0, 28)
		btn.Position = UDim2.new(0, 5, 0, tabY)
		btn.Text = name
		btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
		btn.TextColor3 = currentColor
		btn.Font = Enum.Font.GothamBold
		btn.TextSize = 11
		btn.Parent = sidebarFrame
		Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
		tabY = tabY + 33
		table.insert(tabOrderList, btn)

		local Tab = Instance.new("ScrollingFrame")
		Tab.Size = UDim2.new(1, 0, 1, 0)
		Tab.Visible = false
		Tab.Parent = contentFrame
		Tab.ScrollBarThickness = 4
		Tab.ScrollBarImageColor3 = currentColor
		Tab.BackgroundTransparency = 1
		Tab.BorderSizePixel = 0
		Tab.CanvasSize = UDim2.new(0, 0, 0, 0)
		Tab.ScrollingEnabled = true
		Tab.AutomaticCanvasSize = Enum.AutomaticSize.Y

		local layout = Instance.new("UIListLayout", Tab)
		layout.Padding = UDim.new(0, 8)
		layout.SortOrder = Enum.SortOrder.LayoutOrder
		layout.HorizontalAlignment = Enum.HorizontalAlignment.Center

		local pad = Instance.new("UIPadding", Tab)
		pad.PaddingTop = UDim.new(0, 8)
		pad.PaddingBottom = UDim.new(0, 10)

		btn.MouseButton1Click:Connect(function()
			for _, child in ipairs(sidebarFrame:GetChildren()) do
				if child:IsA("TextButton") then
					child.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
					child.TextColor3 = currentColor
				end
			end
			btn.BackgroundColor3 = currentColor
			btn.TextColor3 = Color3.fromRGB(0, 0, 0)
			if currentTab then currentTab.Visible = false end
			currentTab = Tab
			Tab.Visible = true
		end)

		local Elements = {}

		function Elements:AddSection(sectionTitle)
			local Section = {}
			local sectionFrame = Instance.new("Frame")
			sectionFrame.Size = UDim2.new(1, -12, 0, 0)
			sectionFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
			sectionFrame.BackgroundTransparency = 0.3
			sectionFrame.Parent = Tab
			sectionFrame.AutomaticSize = Enum.AutomaticSize.Y
			Instance.new("UICorner", sectionFrame).CornerRadius = UDim.new(0, 6)

			local stroke = Instance.new("UIStroke", sectionFrame)
			stroke.Color = currentColor
			stroke.Thickness = 1

			local sectionLayout = Instance.new("UIListLayout", sectionFrame)
			sectionLayout.Padding = UDim.new(0, 4)

			local titleLabelSection = Instance.new("TextLabel")
			titleLabelSection.Size = UDim2.new(1, 0, 0, 22)
			titleLabelSection.Text = " " .. sectionTitle
			titleLabelSection.BackgroundTransparency = 1
			titleLabelSection.TextColor3 = currentColor
			titleLabelSection.TextSize = 12
			titleLabelSection.Font = Enum.Font.GothamBold
			titleLabelSection.TextXAlignment = Enum.TextXAlignment.Left
			titleLabelSection.Parent = sectionFrame

			local container = Instance.new("Frame")
			container.Size = UDim2.new(1, 0, 0, 0)
			container.BackgroundTransparency = 1
			container.Parent = sectionFrame
			container.AutomaticSize = Enum.AutomaticSize.Y

			local clayout = Instance.new("UIListLayout", container)
			clayout.Padding = UDim.new(0, 6)
			clayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

			local cpad = Instance.new("UIPadding", container)
			cpad.PaddingBottom = UDim.new(0, 6)

			function Section:AddLabel(text)
				local l = Instance.new("TextLabel")
				l.Size = UDim2.new(1, -12, 0, 20)
				l.Text = text
				l.BackgroundTransparency = 1
				l.TextColor3 = Color3.fromRGB(220, 220, 220)
				l.Font = Enum.Font.Gotham
				l.TextSize = 11
				l.TextXAlignment = Enum.TextXAlignment.Left
				l.TextTruncate = Enum.TextTruncate.AtEnd
				l.Parent = container
				return l
			end

			function Section:AddButton(text, callback)
				local b = Instance.new("TextButton")
				b.Size = UDim2.new(1, -12, 0, 32)
				b.Text = text
				b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
				b.TextColor3 = currentColor
				b.Font = Enum.Font.GothamBold
				b.TextSize = 11
				b.Parent = container
				Instance.new("UICorner", b).CornerRadius = UDim.new(0, 4)
				b.MouseButton1Click:Connect(function()
					if callback then callback() end
				end)
				return b
			end

			function Section:AddToggle(text, defaultOrCb, maybeCb)
				local default = false
				local callback = nil
				if type(defaultOrCb) == "boolean" then
					default = defaultOrCb
					callback = maybeCb
				else
					callback = defaultOrCb
				end

				local state = default == true
				local f = Instance.new("Frame")
				f.Size = UDim2.new(1, -12, 0, 36)
				f.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
				f.Parent = container
				Instance.new("UICorner", f).CornerRadius = UDim.new(0, 4)

				local label = Instance.new("TextLabel")
				label.Size = UDim2.new(1, -55, 1, 0)
				label.Position = UDim2.new(0, 10, 0, 0)
				label.Text = text
				label.BackgroundTransparency = 1
				label.TextColor3 = currentColor
				label.Font = Enum.Font.Gotham
				label.TextSize = 11
				label.TextXAlignment = Enum.TextXAlignment.Left
				label.TextTruncate = Enum.TextTruncate.AtEnd
				label.Parent = f

				local toggle = Instance.new("Frame")
				toggle.Size = UDim2.new(0, 40, 0, 20)
				toggle.Position = UDim2.new(1, -50, 0.5, -10)
				toggle.BackgroundColor3 = state and currentColor or Color3.fromRGB(50, 50, 50)
				toggle.Parent = f
				Instance.new("UICorner", toggle).CornerRadius = UDim.new(1, 0)

				local circle = Instance.new("Frame")
				circle.Size = UDim2.new(0, 16, 0, 16)
				circle.Position = state and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
				circle.BackgroundColor3 = state and Color3.fromRGB(0, 0, 0) or currentColor
				circle.Parent = toggle
				Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)

				f.InputBegan:Connect(function(input)
					if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
						state = not state
						local bgColor = state and currentColor or Color3.fromRGB(50, 50, 50)
						TweenService:Create(toggle, TweenInfo.new(0.2), { BackgroundColor3 = bgColor }):Play()
						TweenService:Create(circle, TweenInfo.new(0.2), {
							Position = state and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8),
							BackgroundColor3 = state and Color3.fromRGB(0, 0, 0) or currentColor
						}):Play()
						if callback then callback(state) end
					end
				end)

				return f
			end

			function Section:AddSlider(text, min, max, defaultOrCb, maybeCb)
				local default = min
				local callback = nil
				if type(defaultOrCb) == "number" then
					default = math.clamp(defaultOrCb, min, max)
					callback = maybeCb
				else
					callback = defaultOrCb
				end

				local value = default
				local dragging = false
				local initAlpha = math.clamp((value - min) / math.max(max - min, 1), 0, 1)

				local frame = Instance.new("Frame")
				frame.Size = UDim2.new(1, -12, 0, 55)
				frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
				frame.Parent = container
				Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 4)

				local label = Instance.new("TextLabel")
				label.Size = UDim2.new(1, -10, 0, 20)
				label.Position = UDim2.new(0, 5, 0, 5)
				label.Text = text .. " (" .. value .. ")"
				label.BackgroundTransparency = 1
				label.TextColor3 = currentColor
				label.Font = Enum.Font.Gotham
				label.TextSize = 11
				label.Parent = frame

				local bar = Instance.new("Frame")
				bar.Size = UDim2.new(1, -20, 0, 5)
				bar.Position = UDim2.new(0, 10, 1, -15)
				bar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
				bar.Parent = frame
				Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)

				local fill = Instance.new("Frame")
				fill.Size = UDim2.new(initAlpha, 0, 1, 0)
				fill.BackgroundColor3 = currentColor
				fill.Parent = bar
				Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

				local function update(input)
					if bar.AbsoluteSize.X <= 0 then return end
					local pos = math.clamp((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
					fill.Size = UDim2.new(pos, 0, 1, 0)
					value = math.floor(min + (max - min) * pos + 0.5)
					label.Text = text .. " (" .. value .. ")"
					if callback then callback(value) end
				end

				bar.InputBegan:Connect(function(input)
					if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
						dragging = true
						update(input)
					end
				end)
				UIS.InputChanged:Connect(function(input)
					if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
						update(input)
					end
				end)
				UIS.InputEnded:Connect(function(input)
					if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
						dragging = false
					end
				end)

				return frame
			end

			return Section
		end

		return Elements
	end

	function windowFunctions:SetThemeColor(color)
		updateThemeColor(color)
	end
	function windowFunctions:SetToggleIcon(iconAssetId)
		if toggleButton then
			toggleButton.Image = "rbxassetid://" .. iconAssetId
		end
	end
	function windowFunctions:SetToggleColor(color)
		if toggleButton then
			toggleButton.BackgroundColor3 = color
		end
	end
	function windowFunctions:SetToggleSize(size)
		updateToggleButtonSize(size)
	end
	function windowFunctions:Notify(titulo, texto, duracao)
		Notify(titulo, texto, duracao)
	end

	local open = true
	local notified = false
	toggleButton.MouseButton1Click:Connect(function()
		if movedBtn then return end
		open = not open
		mainFrame.Visible = open
		if open and not notified then
			Notify("Welcome Sunset 🌞", "99 Nights Loaded 🌞", 4)
			notified = true
		end
	end)

	task.defer(function()
		if tabOrderList[1] then
			firesignal(tabOrderList[1].MouseButton1Click)
		end
	end)

	return windowFunctions
end

--========================================================
-- MOBILE FLY PAD
--========================================================

local flyPad = Instance.new("Frame")
flyPad.Size = UDim2.fromOffset(60, 124)
flyPad.Position = UDim2.new(1, -70, 1, -132)
flyPad.BackgroundTransparency = 1
flyPad.Visible = false
flyPad.Parent = CoreGui.Parent and CoreGui or PlayerGui
pcall(function() flyPad.Parent = CoreGui end)
if flyPad.Parent ~= CoreGui then
	flyPad.Parent = PlayerGui
end

local function holdBtn(yOff, text, onState)
	local b = Instance.new("TextButton")
	b.Size = UDim2.fromOffset(56, 56)
	b.Position = UDim2.fromOffset(0, yOff)
	b.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
	b.Text = text
	b.TextColor3 = Color3.fromRGB(255, 128, 0)
	b.Font = Enum.Font.GothamBold
	b.TextSize = 20
	b.Parent = flyPad
	Instance.new("UICorner", b).CornerRadius = UDim.new(1, 0)
	local st = Instance.new("UIStroke", b)
	st.Color = Color3.fromRGB(255, 128, 0)
	b.MouseButton1Down:Connect(function() onState(true) end)
	b.MouseButton1Up:Connect(function() onState(false) end)
	b.MouseLeave:Connect(function() onState(false) end)
	b.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch then onState(true) end
	end)
	b.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch then onState(false) end
	end)
end
holdBtn(0,  "▲", function(v) padUp   = v end)
holdBtn(68, "▼", function(v) padDown = v end)

--========================================================
-- BUILD MENU & CONFLICT WARNING
--========================================================

local Win = Library:CreateWindow("SUNSET")

local chopWarnLbl = nil
local function refreshChopCombatWarning()
	local bothOn = CFG.KillAura and CFG.AutoChop
	if chopWarnLbl and chopWarnLbl.Parent then
		if bothOn then
			chopWarnLbl.Text = "⚠️ Đang bật cả Kill Aura & Auto Chop (dễ xung đột vũ khí)!"
			chopWarnLbl.TextColor3 = Color3.fromRGB(255, 85, 85)
		else
			chopWarnLbl.Text = "✅ Không xung đột vũ khí"
			chopWarnLbl.TextColor3 = Color3.fromRGB(140, 230, 140)
		end
	end
	if bothOn and Win and Win.Notify then
		Win:Notify(
			"⚠️ Cảnh báo xung đột vũ khí",
			"Đang bật cả Kill Aura và TP Auto Chop. Nên tắt Kill Aura khi farm gỗ để chặt mượt hơn!",
			5
		)
	end
end

-- TAB 1: CHÍNH
local TabMain = Win:CreateTab("Chính")
do
	local sCombat = TabMain:AddSection("COMBAT & FARM")
	local tool = getKillTool()
	local chopTool = getChopTool()
	sCombat:AddLabel("🗡 Vũ khí: " .. (tool and tool.Name or "Không có") .. " | 🪓 Rìu: " .. (chopTool and chopTool.Name or "Không có"))

	sCombat:AddToggle("Kill Aura", CFG.KillAura, function(v)
		CFG.KillAura = v
		refreshChopCombatWarning()
	end)
	sCombat:AddSlider("Kill Radius", 10, 500, CFG.KillRadius, function(v) CFG.KillRadius = v end)

	sCombat:AddToggle("TP Auto Chop (Chặt gãy mới đổi)", CFG.AutoChop, function(v)
		CFG.AutoChop = v
		refreshChopCombatWarning()
	end)
	sCombat:AddSlider("Chop Radius", 10, 500, CFG.ChopRadius, function(v) CFG.ChopRadius = v end)

	chopWarnLbl = sCombat:AddLabel("✅ Không xung đột vũ khí")
	chopWarnLbl.TextColor3 = Color3.fromRGB(140, 230, 140)

	sCombat:AddSlider("Delay (x10ms)", 5, 60, math.floor(CFG.Delay * 100), function(v) CFG.Delay = v / 100 end)

	local sChest = TabMain:AddSection("RƯƠNG, ĐẠN & ĐỒNG XU")
	sChest:AddToggle("Mở rương + nhặt đồ & đạn (2s)", CFG.AutoChestAndLoot, function(v)
		CFG.AutoChestAndLoot = v
		if v then
			task.spawn(function() refreshAmmoCache(true) end)
		else
			updateChestLootStatus(
				string.format("Tắt · Rương: %d | Đồ: %d | Đạn: %d",
					totalChestsOpened, totalItemsLooted, totalAmmoCollected)
			)
		end
	end)
	chestLootStatusLbl = sChest:AddLabel(chestLootStatusText)

	sChest:AddToggle("Tự động nhặt toàn bộ xu", CFG.AutoCollectCoins, function(v)
		CFG.AutoCollectCoins = v
		if v then
			task.spawn(collectAllCoinsOnce)
		else
			updateCoinStatus("Tắt (Tổng: " .. totalCollectedCoins .. ")")
		end
	end)
	coinStatusLbl = sChest:AddLabel(coinStatusText)

	local sSurv = TabMain:AddSection("SINH TỒN & TRỒNG CÂY")
	sSurv:AddToggle("Auto Eat Carrot", CFG.AutoEat, function(v)
		CFG.AutoEat = v
		if not v then updateEatStatus("Tắt") end
	end)
	eatStatusLbl = sSurv:AddLabel(eatStatusText)

	local plantLbl = sSurv:AddLabel("Đứng chỗ trồng → bấm nút")
	sSurv:AddButton("🌱 Trồng tất cả mầm cây", function()
		task.spawn(function() plantAllSaplings(plantLbl) end)
	end)
end

-- TAB 2: DI CHUYỂN
local TabMove = Win:CreateTab("Di chuyển")
do
	local sMove = TabMove:AddSection("MOVEMENT")
	sMove:AddToggle("Speed", CFG.Speed, function(v)
		CFG.Speed = v
		if not v then local h = hum(); if h then h.WalkSpeed = 16 end end
	end)
	sMove:AddSlider("Walk Speed", 16, 300, CFG.WalkSpeed, function(v) CFG.WalkSpeed = v end)

	sMove:AddToggle("High Jump", CFG.Jump, function(v)
		CFG.Jump = v
		if not v then local h = hum(); if h then h.JumpPower = 50 end end
	end)
	sMove:AddSlider("Jump Power", 50, 500, CFG.JumpPower, function(v) CFG.JumpPower = v end)

	sMove:AddToggle("Fly", CFG.Fly, function(v)
		CFG.Fly = v
		flyPad.Visible = v
		if v then startFly() else stopFly() end
	end)
	sMove:AddSlider("Fly Speed", 10, 200, CFG.FlySpeed, function(v) CFG.FlySpeed = v end)

	sMove:AddToggle("Noclip", CFG.Noclip, function(v)
		CFG.Noclip = v
		if not v then restoreNoclip() end
	end)
end

-- TAB 3: TRẺ EM
local TabChild = Win:CreateTab("Trẻ em")
do
	local sAutoRescue = TabChild:AddSection("AUTO CỨU TRẺ VỀ LỬA TRẠI")
	local sack = getSackTool()
	sAutoRescue:AddLabel("🎒 Túi hiện tại: " .. (sack and sack.Name or "Chưa có túi"))

	sAutoRescue:AddToggle("Auto cứu trẻ (TP → Balo → Lửa)", CFG.AutoRescueChild, function(v)
		CFG.AutoRescueChild = v
		if not v then
			updateRescueStatus(string.format("Tắt · Đã cứu: %d", totalChildrenSaved))
		end
	end)
	rescueStatusLbl = sAutoRescue:AddLabel(rescueStatusText)

	sAutoRescue:AddButton("🔄 Reset danh sách trẻ đã cứu", function()
		rescuedChildren  = setmetatable({}, { __mode = "k" })
		rescuedChildKeys = {}
		updateRescueStatus("🔄 Đã reset danh sách trẻ đã cứu")
	end)

	local sChild = TabChild:AddSection("ESP & TP THỦ CÔNG")
	sChild:AddToggle("ESP Trẻ em", CFG.ESPChild, function(v)
		CFG.ESPChild = v
		if not v then clearESPChild() end
	end)

	local r, list = root(), {}
	for m, _ in pairs(childSet) do
		if m and m.Parent then
			local p = partOf(m)
			if p then
				local d = r and math.floor((p.Position - r.Position).Magnitude) or 0
				table.insert(list, {m=m, p=p, d=d})
			end
		end
	end
	table.sort(list, function(a, b) return a.d < b.d end)

	if #list == 0 then
		sChild:AddLabel("Không tìm thấy trẻ nào")
	else
		for i, item in ipairs(list) do
			sChild:AddButton(string.format("TP %d. %s (%d studs)", i, item.m.Name, item.d), function()
				tpTo(item.m)
			end)
		end
	end
end

-- TAB 4: TP & ĐỒ
local TabItems = Win:CreateTab("TP & Đồ")
do
	local sFire = TabItems:AddSection("LỬA TRẠI & NHIÊN LIỆU")
	sFire:AddButton("🔥 TP Lửa trại", tpCampfire)
	sFire:AddToggle("Auto dồn nhiên liệu → Lửa", CFG.AutoDrop, function(v) CFG.AutoDrop = v end)
	sFire:AddButton("🪵 Dồn nhiên liệu → Lửa ngay", function()
		task.spawn(function() dumpFuelToFire() end)
	end)

	local sPull = TabItems:AddSection("KÉO VẬT PHẨM VỀ NGƯỜI")
	sPull:AddButton("🍎 Kéo thức ăn", function()
		task.spawn(function() pullAllExact(FOOD_ITEMS) end)
	end)
	sPull:AddButton("⚙️ Kéo nguyên liệu (Log, Chair...)", function()
		task.spawn(function() pullAllExact(SCRAP_ITEMS) end)
	end)

	local sPlr = TabItems:AddSection("NGƯỜI CHƠI")
	local r = root()
	local hasOther = false
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= Player then
			hasOther = true
			local pRoot = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
			local dist = (r and pRoot) and math.floor((pRoot.Position - r.Position).Magnitude) or 0
			sPlr:AddButton(string.format("Kéo %s (%d studs)", plr.Name, dist), function()
				pullPlayerToMe(plr)
			end)
		end
	end
	if not hasOther then
		sPlr:AddLabel("Không có người chơi khác")
	end
end

-- TAB 5: CÀI ĐẶT
local TabSet = Win:CreateTab("Cài đặt")
do
	local sPerf = TabSet:AddSection("HIỆU NĂNG & ÁNH SÁNG")
	sPerf:AddToggle("Fix Lag Pro", CFG.FixLag, function(v) setFixLag(v) end)
	sPerf:AddToggle("Nhìn trong bóng tối (Fullbright)", CFG.Fullbright, function(v)
		CFG.Fullbright = v
		setFullbright(v)
	end)

	local sMap = TabSet:AddSection("BẢN ĐỒ & GIAO DIỆN")
	sMap:AddButton("🗺️ Reveal Map", function()
		task.spawn(revealMap)
	end)
	sMap:AddSlider("Kích thước nút bật/tắt", 32, 70, 40, function(v)
		Win:SetToggleSize(v)
	end)
end

print("========================================")
print(" SUNSET UI + 99 NIGHTS COMPLETE LOADED ✓")
print("========================================")
