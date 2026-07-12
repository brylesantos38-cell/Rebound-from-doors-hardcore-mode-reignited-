game.ReplicatedStorage.GameData.LatestRoom.Changed:Wait()

local function GetRoom()
	local currentRooms = workspace:FindFirstChild("CurrentRooms")
	if not currentRooms then return nil end

	local gameData = game.ReplicatedStorage:FindFirstChild("GameData")
	local latestRoom = gameData and gameData:FindFirstChild("LatestRoom")

	if latestRoom then
		return currentRooms:FindFirstChild(tostring(latestRoom.Value))
	end
	return nil
end

-- Tải Asset an toàn (pcall giúp script không bị dừng nếu lỗi ID)
local success, assets = pcall(function()
	return game:GetObjects("rbxassetid://134261174233456")
end)

if not success or not assets or #assets == 0 then
	warn("⚠️ Không thể tải được Asset! Kiểm tra lại ID hoặc bạn có đang dùng Executor/Plugin không.")
	return
end

local s = assets[1]
s.Name = "Singularity" -- Đổi tên để tránh lỗi Infinite Yield ở đoạn sau
s.Parent = workspace

-- Tìm kiếm entity "Singu" bên trong asset
local entity = s:FindFirstChild("Singu") or s
if not entity then
	warn("⚠️ Không tìm thấy 'Singu' trong Asset!")
	return
end

local room = GetRoom()
if not room then
	warn("⚠️ Không tìm thấy phòng hiện tại từ LatestRoom!")
	return
end

local entrance = room:WaitForChild("RoomEntrance", 5)
if not entrance then
	warn("⚠️ Không tìm thấy vị trí 'RoomEntrance' trong phòng!")
	return
end

-- Dịch chuyển Entity đến trước cửa phòng (Hỗ trợ cả Model lẫn Part)
local spawnCFrame = entrance.CFrame * CFrame.new(0, 0, -5)
if entity:IsA("Model") then
	entity:PivotTo(spawnCFrame)
else
	entity.CFrame = spawnCFrame
end

-- Hàm phát âm thanh an toàn (tránh lỗi nếu thiếu sound)
local function playSound(soundName)
	local sound = entity:FindFirstChild(soundName)
	if sound and sound:IsA("Sound") then
		sound:Play()
	end
end

playSound("Scare")
playSound("caught")
playSound("ScreamingInPublicBathroomsPrank")

task.wait(3)

local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid", 5)
local hrp = char:WaitForChild("HumanoidRootPart", 5)

if hum and hrp then
	local entityPos = entity:IsA("Model") and entity:GetPivot().Position or entity.Position
	local distance = (hrp.Position - entityPos).Magnitude

	local TweenService = game:GetService("TweenService")
	local Lighting = game:GetService("Lighting")
	local targetCFrame = entity:IsA("Model") and entity:GetPivot() or entity.CFrame

	if distance >= 25 then
		local targets = {}
		local processedModels = {}
		local processedParts = {}

		local overlapParams = OverlapParams.new()
		overlapParams.FilterType = Enum.RaycastFilterType.Exclude
		overlapParams.FilterDescendantsInstances = {s, char} -- Loại trừ quái và người chơi

		local nearbyParts = workspace:GetPartBoundsInRadius(entityPos, 60, overlapParams)

		for _, part in ipairs(nearbyParts) do
			local nameLower = string.lower(part.Name)
			if nameLower:find("floor") or nameLower:find("wall") or part.Locked then
				continue
			end

			local parent = part.Parent
			if parent and parent:IsA("Model") and parent ~= workspace then
				if not processedModels[parent] then
					processedModels[parent] = true
					local modelPos = parent:GetPivot().Position
					local dist = (modelPos - entityPos).Magnitude
					table.insert(targets, {instance = parent, distance = dist, isModel = true})
				end
			else
				if not processedParts[part] then
					processedParts[part] = true
					local dist = (part.Position - entityPos).Magnitude
					table.insert(targets, {instance = part, distance = dist, isModel = false})
				end
			end
		end

		table.sort(targets, function(a, b)
			return a.distance < b.distance
		end)

		local maxPull = math.min(#targets, 6)
		local instancesToDestroy = {}

		for i = 1, maxPull do
			local target = targets[i]
			table.insert(instancesToDestroy, target.instance)

			if target.isModel then
				for _, p in ipairs(target.instance:GetDescendants()) do
					if p:IsA("BasePart") then
						p.Anchored = true
						p.CanCollide = false
						TweenService:Create(
							p, 
							TweenInfo.new(1.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
							{CFrame = targetCFrame}
						):Play()
					end
				end
			else
				target.instance.Anchored = true
				target.instance.CanCollide = false
				TweenService:Create(
					target.instance, 
					TweenInfo.new(1.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
					{CFrame = targetCFrame}
				):Play()
			end
		end

		task.delay(0.9, function()
			for _, inst in ipairs(instancesToDestroy) do
				if inst and inst.Parent then
					inst:Destroy()
				end
			end
		end)

wait(0.9)

		local chain = Instance.new("ColorCorrectionEffect", game.Lighting) 
		game.Debris:AddItem(chain, 24)
		chain.Name = "Warn"
		chain.TintColor = Color3.fromRGB(85, 0, 0) 
		chain.Saturation = -0.7 
		chain.Contrast = 0.2

		game.TweenService:Create(chain, TweenInfo.new(15), {TintColor = Color3.fromRGB(255, 255, 255), Saturation = 0, Contrast = 0}):Play()

		local TW = TweenService:Create(game.Lighting.MainColorCorrection, TweenInfo.new(5), {TintColor = Color3.fromRGB(255, 255, 255)})
		TW:Play()

		local CameraShaker = require(game.ReplicatedStorage:WaitForChild("CameraShaker"))
		local camera = workspace.CurrentCamera
		local camShake = CameraShaker.new(Enum.RenderPriority.Camera.Value, function(shakeCf)
			camera.CFrame = camera.CFrame * shakeCf
		end)
		camShake:Start()
		camShake:ShakeOnce(30, 6.5, 0.1, 1, 0.1, 0.5)

		playSound("bloood")
		playSound("gore0")
		playSound("gore1")
		playSound("gore2")
		playSound("gore3")

		if entity:FindFirstChild("gore0") then entity.gore0.Volume = 3 end
		if entity:FindFirstChild("gore1") then entity.gore1.Volume = 3 end
		if entity:FindFirstChild("gore2") then entity.gore2.Volume = 3 end
		if entity:FindFirstChild("gore3") then entity.gore3.Volume = 3 end

	elseif distance <= 25 then
		if entity:IsA("Model") then
			if entity.PrimaryPart then entity.PrimaryPart.Anchored = true end
		else
			entity.Anchored = true
		end

		local tween = TweenService:Create(
			hrp,
			TweenInfo.new(1.2, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
			{CFrame = targetCFrame}
		)
		tween:Play()

		task.wait(0.9)

		local chain = Instance.new("ColorCorrectionEffect", game.Lighting) 
		game.Debris:AddItem(chain, 24)
		chain.Name = "Warn"
		chain.TintColor = Color3.fromRGB(85, 0, 0) 
		chain.Saturation = -0.7 
		chain.Contrast = 0.2

		game.TweenService:Create(chain, TweenInfo.new(15), {TintColor = Color3.fromRGB(255, 255, 255), Saturation = 0, Contrast = 0}):Play()

		local TW = TweenService:Create(game.Lighting.MainColorCorrection, TweenInfo.new(5), {TintColor = Color3.fromRGB(255, 255, 255)})
		TW:Play()

		local CameraShaker = require(game.ReplicatedStorage:WaitForChild("CameraShaker"))
		local camera = workspace.CurrentCamera
		local camShake = CameraShaker.new(Enum.RenderPriority.Camera.Value, function(shakeCf)
			camera.CFrame = camera.CFrame * shakeCf
		end)
		camShake:Start()
		camShake:ShakeOnce(30, 6.5, 0.1, 1, 0.1, 0.5)

		playSound("bloood")
		playSound("gore0")
		playSound("gore1")
		playSound("gore2")
		playSound("gore3")
		hum.Health -= 60
	else
		task.wait(0.9)
	end
end

	task.wait(3)
s:Destroy()
