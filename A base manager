-- This is Base Manager for steal a youtuber RNG by: Jozinn93 on roblox, masternamedjoe on discord
local Server_cesty = require(game:GetService("ServerStorage").Server_cesty)
local Teleport_character_module = require(Server_cesty.Local_moduly.Teleport_character)
local Data_manager = require(Server_cesty.Server_moduly.Data_manager)
local Gate_manager = require(Server_cesty.Server_moduly.Gates_manager)
local Info_animations = require(Server_cesty.Replicated_storage.Moduly_info.Animations)
local Format_number = require(Server_cesty.Replicated_storage.Moduly.Format_number)
local Base_manager = {}
Base_manager.__index = Base_manager

local Bases_spawns = {
	["Base1"] = CFrame.new(59.1, 3.612, -164),
	["Base2"] = CFrame.new(-40.9, 3.612, -164),
	["Base3"] = CFrame.new(-240.9, 3.612, -164),
	["Base4"] = CFrame.new(-240.701, 3.612, -95.863),
	["Base5"] = CFrame.new(-140.701, 3.612, -95.863),
	["Base6"] = CFrame.new(59.299, 3.612, -95.863),
	["Base7"] = CFrame.new(-40.701, 3.612, -95.863),
	["Base8"] = CFrame.new(-140.9, 3.612, -164),
}

local Bases_owned = {
	["Base1"] = "nil",
	["Base2"] = "nil",
	["Base3"] = "nil",
	["Base4"] = "nil",
	["Base5"] = "nil",
	["Base6"] = "nil",
	["Base7"] = "nil",
	["Base8"] = "nil",
}

function Base_manager.Free_base(Player) -- Finds free base and give it to player
	repeat
		task.wait()
	until not Base_manager.Finding

	Base_manager.Finding = true
	 
	for base_name, owns in pairs(Bases_owned) do 
		if owns == "nil" then
			Teleport_character_module.Teleport(Player.Character, Bases_spawns[base_name])
			
			local Base = workspace.Map.Bases:WaitForChild(base_name)
			Base:SetAttribute("Owning_player", Player.Name)
			Bases_owned[Base.Name] = Player
			Base_manager.Finding = nil
			return Base
		end
	end
	Base_manager.Finding = nil
end

function Base_manager.Get_player_base(Player)
	return Base_manager[Player]
end

function Base_manager.New_base(Base,Player)
	local function Requirements()
		if not Base or not Player then return end 
		if not Base:IsA("Folder") or not Player:IsA("Player") then return end 

		return true
	end

	if Requirements() then
		local self = setmetatable({}, Base_manager)
		Base_manager[Player] = self 

		self.Base_player = Player
		self.Base = Base
		self.Base_spawn = Base.Spawn
		self.Base_countdown_text = "This base can be locked in:"
		self.Collect_part = Base.Collect_part
		self.Spawn_cframe = Bases_spawns[Base.Name]
		self.Stealings = 0
		
		local function Collect_part_touch_effect(Collect_part_model)
			local Mesh = Collect_part_model["Meshes/Untitled"]:Clone()
			Mesh.Parent = Collect_part_model
			Mesh.Name = "Effect"
			
			local Highlight = Instance.new("Highlight", Mesh)
			Highlight.FillColor = Color3.fromRGB(77, 255, 0)
			Highlight.OutlineColor = Color3.fromRGB(10, 103, 0)
			
			local tween:Tween = Server_cesty.Tween_service:Create(Mesh, TweenInfo.new(0.75, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {Size = Mesh.Size + Vector3.new(4,0,4), Transparency = 1})
			tween:Play()
			tween.Completed:Once(function()
				Mesh:Destroy()
			end)
		end
		
		self.Check_floor = function(Wait) -- If player base have second floor it will show floor2 models
			local player_data = Data_manager.GetData(Player)
			if player_data.Floor >= 2 then
				local function Set(item)
					if item:IsA("BasePart") then
						if item.Name ~= "Gate_status_part_text" then
						if item:GetAttribute("Glass") then
							item.Transparency = 0.85
						else
							item.Transparency = 0
						end

						item.CanCollide = true
						end
					elseif item:IsA("SpotLight") then
						item.Enabled = true
					end
				end
				
				local function Unlock_floor()
					for i,v in pairs(self.Base["2floor"]:GetDescendants()) do 
						Set(v)
					end
				end
				
				Unlock_floor()
				
				if Wait then
					local a : BasePart = self.Base["2floor"]
					a.DescendantAdded:Connect(function(Child)
						Set(Child)
					end)
				end
			end
		end
		self.Check_floor(true)
		
		self.Give_back_stealed_brainrot = function(Owning_player)
			for index, Youtuber_info in pairs(Data_manager.GetData(Player).Youtubers) do 
				if Youtuber_info.Owner then
					local Owner_player = Server_cesty.Players:FindFirstChild(Youtuber_info.Owner)
					if Owner_player then
						local Owner_base = Base_manager.Get_player_base(Owner_player)
						if Owning_player then
							if Owning_player ~= Owner_player then
								continue
							end
						end
						
						local Owner_base = Base_manager.Get_player_base(Owner_player)
						if Owner_base.Stealings and Owner_base then
							Owner_base.Stealings -= 1
							if Owner_base.Stealings <= 0 then
								Owner_base.Stealings = 0
							end
							if Owner_base.I_stealed then
								Owner_base.I_stealed:Destroy()

								Owner_base:Load_base()
							end
						end
						
						Owner_base:Add_youtuber({Youtuber_name = Youtuber_info.Youtuber_name, Youtuber_specialid = Youtuber_info.Youtuber_specialid})
						self:Remove_youtuber({Youtuber_specialid = Youtuber_info.Youtuber_specialid})

						Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Player, {Type = "Collect_zone", Enabled = false, Base = Base})
						self:Load_base()
					end
				end
			end
		end
		
		self.Collect_part_touch_connection = self.Collect_part.Touch_part.Touched:Connect(function(Hit)
			local Hit_player = game.Players:GetPlayerFromCharacter(Hit.Parent)
			if Hit_player then
				if not self.Touch_cooldown and Hit_player == Player then
					self.Touch_cooldown = true
					
					task.delay(.5, function()
						self.Touch_cooldown = false
					end)
					
					local function Destroy_stealed_model_youtuber()
						for index, youtuber in pairs(Player.Character:GetChildren()) do 
							if youtuber:GetAttribute("Stealed_youtuber") then
								youtuber:Destroy()
							end
						end
					end
					
					local function Steal_brainrot()
						for index, Youtuber_info in pairs(Data_manager.GetData(Player).Youtubers) do 
							if Youtuber_info.Owner then
								local Owner_player = Server_cesty.Players:FindFirstChild(Youtuber_info.Owner)
								if Owner_player then
									Collect_part_touch_effect(self.Collect_part)
									Data_manager.GetData(Player).Youtubers[index].Owner = nil
									Destroy_stealed_model_youtuber()
									Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Player, {Type = "Collect_zone", Enabled = false, Base = Base, Stealed = true})
									self:Load_base()
								end
							end
						end
					end
					
					Steal_brainrot()
				end
			end
		end)
		
		

		self.Find_youtuber_table_by_specialid = function(Table)
			for index, Player_data_youtuber_table in pairs(Table.Data.Youtubers) do 
				if Player_data_youtuber_table.Youtuber_specialid == Table.Youtuber_specialid then
					return Player_data_youtuber_table, index
				end
			end
		end

		self.Is_base_full = function()
			local Base_player_data = Data_manager.GetData(Player)
			local Count_of_youtubers = #Base_player_data.Youtubers
			local Base_player_data_unlocked_floor = Base_player_data.Floor
			
			if self.Stealings then
				Count_of_youtubers += self.Stealings
			end
		
			if Base_player_data_unlocked_floor <= 1 then
				if Count_of_youtubers >= 6 then
					return true
				end
			elseif Base_player_data_unlocked_floor >= 2 then
				if Count_of_youtubers >= 11 then
					return true
				end
			end
		end
		
		self.Give_youtuber_in_hands = function(Youtuber, Character)
			local Cloned = Youtuber:Clone()
			Cloned.Parent = Character
			Cloned:SetAttribute("Stealed_youtuber", true)
			self.I_stealed = Cloned
			
			local function Set_character_massless()
				for i,v in pairs(Cloned:GetDescendants()) do 
					if v:IsA("BasePart") then
						v.Massless = true
					end
				end
			end
			
			local function Find_humanoid()
				for i,v in pairs(Cloned:GetDescendants()) do 
					if v:IsA("Humanoid") then
						return v 
					end
				end
			end

			local function Set_collision_group()
				for i,v in pairs(Cloned:GetDescendants()) do 
					if v:IsA("BasePart") then
						v.CollisionGroup = "Stealed"
						v.Anchored = false
					end
				end
			end

			local function Remove_all_proximity()
				for i,v in pairs(Cloned:GetDescendants()) do 
					if v:IsA("ProximityPrompt") then
						v:Destroy()
					end
				end
			end

			Set_collision_group()
			local Cloned_humanoid = Find_humanoid()
			Cloned_humanoid.PlatformStand = true

			local Weld = Instance.new("Weld", Cloned.PrimaryPart)
			Weld.Part0 = Cloned.PrimaryPart
			Weld.Part1 = Character.HumanoidRootPart
			Weld.Name = "Stealing_Weld"

			Weld.C0 = CFrame.new(0,0,1.75)

			Remove_all_proximity()
			Set_character_massless()
			return Cloned
		end
		
		Base:SetAttribute("Locked", true)
		Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Player, {Type = "Base_manager_local", Base = Base})
		
		Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireAllClients({Type = "Base_ui_others"})
		return self
	end
end

function Base_manager:Teleport_to_spawn()
	local Base_player = self.Base_player
	local Base_character = Base_player.Character
	local Base_spawn = self.Spawn_cframe

	if Base_player and Base_spawn and Base_character then
		Teleport_character_module.Teleport(Base_character, Base_spawn)
	end
end

function Base_manager:Remove_npcs() -- Delete youtubers from base
	for I, Youtuber in pairs(self.Base:GetDescendants()) do 
		if Youtuber:GetAttribute("Youtuber") then
			Youtuber:Destroy()
		end
		if Youtuber.Name == "Collector" then
			if self[Youtuber] then
				self[Youtuber]:Disconnect()
				self[Youtuber] = nil
			end
			Youtuber:Destroy()
		end
		if Youtuber.Name == "Sit_part" then
			if Youtuber:GetAttribute("Sitting_youtuber") then
				Youtuber:SetAttribute("Sitting_youtuber", nil)
			end
		end
	end
end

function Base_manager:Remove() -- When player is leaving game
	local Base_player = self.Base_player

	if Base_manager[Base_player] then
		for index, _player in pairs(Server_cesty.Players:GetPlayers()) do 
			if _player ~= Base_player then
				Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(_player, {Type = "Player_leaving_base", Base = self.Base})
			end
		end
		for base_name, owner in pairs(Bases_owned) do 
			if owner == Base_player then
				Bases_owned[base_name] = "nil"
			end
		end
		
		self.Give_back_stealed_brainrot()
		self:Cancel_moving_youtuber()
		
		for index, connection in pairs(Base_manager[Base_player]) do 
			if typeof(connection) == "RBXScriptConnection" then
				connection:Disconnect()
			end
			if typeof(connection) == "thread" then
				task.cancel(connection)
			end
		end
		
		for index, collector in pairs(self.Base:GetDescendants()) do 
			if collector.Name == "Collector" then
				if self[collector] then
					self[collector]:Disconnect()
				end
				collector:Destroy()
			end
		end
		
		if Base_manager[Base_player]["Gate_new"] then
			Base_manager[Base_player]["Gate_new"]:Open()
		end
		self:Remove_npcs()
		Base_manager[Base_player] = nil
	end

	setmetatable(self, nil)
end

function Base_manager:Cancel_moving_youtuber()
	local Base_player = self.Base_player
	local Base = self.Base 
	if Base and Base_player and self.Moving_index then
		local Base_player_data = Data_manager.GetData(Base_player)
		Base_player_data.Youtubers[self.Moving_index].Moving = nil
		self.Moving_youtuber_model:Destroy()
		
		Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player, {Type = "Delete_Move_parts"})
		
		self.Moving_index = nil
		self:Load_base()
	end
end

function Base_manager:Place_youtuber(Youtuber_specialid, Place) -- When player place youtuber after moving him in base
	local Base_player = self.Base_player
	local Base = self.Base 
	if Base and Base_player then
		if self.Moving_index then
			local Base_player_data = Data_manager.GetData(Base_player)
			local Player_youtuber_table_specific, Returned_index = self.Find_youtuber_table_by_specialid({Data = Base_player_data, Youtuber_specialid = self.Moving_specialid})
			
			local function Get_youtuber_of_place()
				for index, youtuber in pairs(Place:GetChildren()) do 
					if youtuber:GetAttribute("Youtuber") then
						return youtuber
					end
				end
			end
			
			local Youtuber_of_place = Get_youtuber_of_place()
			if not Youtuber_of_place then
				for index, child in pairs(self.Base:GetDescendants()) do 
					if child.Name == Player_youtuber_table_specific.Youtuber_location then
						local Collector = child:FindFirstChild("Collector") 
						if Collector then
							if self[Collector] then
								self[Collector]:Disconnect()
							end
							Collector:Destroy()
						end
					end
				end
				
				Player_youtuber_table_specific.Youtuber_location = Place.Name
				Player_youtuber_table_specific.Youtuber_floor = Place.Parent.Name

				self:Load_base_one(Base_player_data.Youtubers[Returned_index])
			else
				local Youtuber_of_place_special_id = Youtuber_of_place:GetAttribute("Youtuber_specialid")
				local Youtuber_of_place_table, Youtuber_of_place_index = self.Find_youtuber_table_by_specialid({Data = Base_player_data, Youtuber_specialid = Youtuber_of_place_special_id})
				
				local Youtuber_of_place_table_location = Youtuber_of_place_table.Youtuber_location
				local Youtuber_of_place_table_floor = Youtuber_of_place_table.Youtuber_floor
				
				local Player_youtuber_table_specific_place = Player_youtuber_table_specific.Youtuber_location
				local Player_youtuber_table_specific_place_floor = Player_youtuber_table_specific.Youtuber_floor
				
				for index, child in pairs(self.Base:GetDescendants()) do 
					if child.Name == Youtuber_of_place_table_location then
						local Collector = child:FindFirstChild("Collector") 
						if self[Collector] then
							self[Collector]:Disconnect()
						end
						if Collector then
							Collector:Destroy()
						end
					end
					if child.Name == Player_youtuber_table_specific.Youtuber_location then
						local Collector = child:FindFirstChild("Collector") 
						if self[Collector] then
							self[Collector]:Disconnect()
						end
						if Collector then
							Collector:Destroy()
						end
					end
				end
				
				Youtuber_of_place_table.Youtuber_location = Player_youtuber_table_specific_place
				Youtuber_of_place_table.Youtuber_floor = Player_youtuber_table_specific_place_floor
				
				Player_youtuber_table_specific.Youtuber_location = Youtuber_of_place_table_location
				Player_youtuber_table_specific.Youtuber_floor = Youtuber_of_place_table_floor
				
				self:Load_base_one(Base_player_data.Youtubers[Returned_index])
				self:Load_base_one(Base_player_data.Youtubers[Youtuber_of_place_index])
				
				Youtuber_of_place:Destroy()
			end

			Base_player_data.Youtubers[self.Moving_index].Moving = nil
			self.Moving_youtuber_model:Destroy()
			Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player, {Type = "Delete_Move_parts"})
			
			self.Moving_index = nil
			self.Moving_youtuber_model = nil
		end
	end
end

function Base_manager:Move_youtuber(Youtuber, Youtuber_specialid) -- Player move youtuber
	local Base_player = self.Base_player
	local Base = self.Base 
	if Base and Base_player and not self.Moving_index then
		local function Delete_origo_youtuber()
			for index, youtuber in pairs(Base:GetDescendants()) do 
				if youtuber:GetAttribute("Youtuber_specialid") then
					if youtuber:GetAttribute("Youtuber_specialid") == Youtuber_specialid then
						youtuber:Destroy()
					end
				end
			end
		end
		
		local Base_player_data = Data_manager.GetData(Base_player)
		local Player_youtuber_table_specific, Returned_index = self.Find_youtuber_table_by_specialid({Data = Base_player_data, Youtuber_specialid = Youtuber_specialid})
		
		self.Moving_youtuber_model = self.Give_youtuber_in_hands(Youtuber, Base_player.Character)
		self.Moving_specialid = Youtuber_specialid
		Base_player_data.Youtubers[Returned_index].Moving = true
		
		Delete_origo_youtuber()
		
		self.Moving_index = Returned_index
		return true
	end
end

function Base_manager:Add_youtuber(Youtuber_information) -- Adding youtuber to player base
	local Base_player = self.Base_player
	local Base = self.Base 

	if Base and Base_player then
		local Player_data = Data_manager.GetData(Base_player)
		if self.Is_base_full() then 
			Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player,{Type = "Notification",Message = "Your base don't have space for another youtuber"})
			return 
		end
		
		local function UnlockedFloor(Floor_name)
			local string_splitted = string.split(Floor_name, "")
			for index, word in pairs(string_splitted) do 
				if tonumber(word) then
					if Player_data.Floor >= tonumber(word) then
						return true
					end
				end
			end
		end

		local function Different()
			for index, floor in pairs(Base:GetChildren()) do 
				for index1_5, floor_child in ipairs(floor:GetChildren()) do 
					if floor_child:IsA("Folder") then
						for index2, place_item in ipairs(floor_child:GetChildren()) do 
							if place_item.Name == "Sit_part" then
								if not place_item:GetAttribute("Sitting_youtuber") and UnlockedFloor(floor.Name) then
									return floor_child, floor
								end
							end
						end
					end
				end
			end
		end
		
		local function Get_free_youtuber_place()
			for index, floor in pairs(Base:GetChildren()) do 
				for index1_5, floor_child in ipairs(floor:GetChildren()) do 
					if floor_child:IsA("Folder") then
						local splitted_word = string.split(floor_child.Name, "")
						for index, word in pairs(splitted_word) do 
							local numbered_word = tonumber(word)
							if numbered_word then
								local to_find = tostring(#Player_data.Youtubers + 1)
								if string.find(floor_child.Name, to_find) then
									for index2, place_item in ipairs(floor_child:GetChildren()) do 
										if place_item.Name == "Sit_part" then
											if not place_item:GetAttribute("Sitting_youtuber") and UnlockedFloor(floor.Name) then
												return floor_child, floor
											end
										end
									end
								end
							end
						end
					end
				end
			end
			return Different()
		end
		
		local function Set_free_location_and_floor()
			local Free_place, Floor = Get_free_youtuber_place()
			if Free_place and Floor then
				if not Youtuber_information.Youtuber_location then
					Youtuber_information.Youtuber_location = Free_place.Name
				end
				if not Youtuber_information.Youtuber_floor then
					Youtuber_information.Youtuber_floor = Floor.Name
				end
				return true
			else
				Free_place, Floor = Different()
				if not Free_place or not Floor then
					return
				end

				if not Youtuber_information.Youtuber_location then
					Youtuber_information.Youtuber_location = Free_place.Name
				end
				if not Youtuber_information.Youtuber_floor then
					Youtuber_information.Youtuber_floor = Floor.Name
				end
				return true
			end
		end
		
		if not Set_free_location_and_floor() then print("No floor or place for youtuber found") return end 
		
		if not Youtuber_information.Youtuber_specialid then
			Youtuber_information.Youtuber_specialid = bit32.byteswap(Random.new():NextInteger(1, 999999999))
		end 
		
		if not Youtuber_information.Earnings then
			Youtuber_information.Earnings = 0
		end
		
		table.insert(Player_data.Youtubers, Youtuber_information)

		if not Youtuber_information.Cancel_load and not Youtuber_information.Owner then
			self:Load_base()
		end
	end
end

function Base_manager:Remove_youtuber(Table)
	local Base_player = self.Base_player
	local Base = self.Base 

	if Table.Youtuber_specialid then
		local Player_data = Data_manager.GetData(Base_player)
		local function Remove_attribute_in_chair()
			for index, part in pairs(Base:GetDescendants()) do 
				--if part.Name == "Sit_part" then
					if part:GetAttribute("Sitting_youtuber") then
						if part:GetAttribute("Sitting_youtuber") == Table.Youtuber_specialid then
							part:SetAttribute("Sitting_youtuber", nil)
						end
					end
				--end
			end
		end

		local Possible_youtuber_data_table, index = self.Find_youtuber_table_by_specialid({Data = Player_data, Youtuber_specialid = Table.Youtuber_specialid})
		if Possible_youtuber_data_table then
			
			for index, child in pairs(Base:GetDescendants()) do 
				if child.Name == Possible_youtuber_data_table.Youtuber_location then
					local Possible_collector = child:FindFirstChild("Collector")
					if Possible_collector then
						if self[Possible_collector] then
							self[Possible_collector]:Disconnect()
						end
						Possible_collector:Destroy()
						print("deteling, collector")
					end
				end
			end
			
			table.remove(Player_data.Youtubers, index)
			if Table.Youtuber_model then
				Table.Youtuber_model:Destroy()
			end

			Remove_attribute_in_chair()
		end		
	end
end

function Base_manager:Steal_youtuber(Table) -- Other players can steal youtuber from player base
	if not Table then return end 
	
	local Base_player = self.Base_player
	local Base = self.Base 
	if Base and Base_player and Table.Stealing_player then
		local function Is_stealing()
			for index, youtuber in pairs(Table.Stealing_player.Character:GetChildren()) do 
				if youtuber:GetAttribute("Stealed_youtuber") then
					return true
				end
			end
		end
		if Is_stealing() then return end 
		
		local Stealed_Base = Base_manager.Get_player_base(Table.Stealing_player)
		
		if Stealed_Base then 
			if Stealed_Base.Is_base_full() then
			Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Table.Stealing_player,{Type = "Notification",Message = "Your base don't have space for another youtuber"})
			return 
			end
		end
		
		local Base_player_data = Data_manager.GetData(Base_player)
		local Player_youtuber_table_specific, Returned_index = self.Find_youtuber_table_by_specialid({Data = Base_player_data, Youtuber_specialid = Table.Youtuber_specialid})
		
		self.Stealings = self.Stealings + 1
	
		local function Remove_stealed_youtuber_from_origo_player()
			for index, child in pairs(Base:GetDescendants()) do 
				if child.Name == Player_youtuber_table_specific.Youtuber_location then
					local Possible_collector = child:FindFirstChild("Collector")
					if Possible_collector then
						if self[Possible_collector] then
							self[Possible_collector]:Disconnect()
						end
						Possible_collector:Destroy()
					end
				end
			end
			
			self:Remove_youtuber({Youtuber_specialid = Table.Youtuber_specialid, Youtuber_model = Table.Youtuber_model})
		end

		local function Add_stealed_youtuber_to_stealing_player()
			if Stealed_Base then
				Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Table.Stealing_player, {Type = "Collect_zone", Enabled = true, Base = Stealed_Base.Base})
				Stealed_Base:Add_youtuber({Owner = Base_player.Name,Youtuber_name = Player_youtuber_table_specific.Youtuber_name, Cancel_load = true, Youtuber_specialid = Table.Youtuber_specialid})
				if self[Table.Stealing_player.Name .. "Died"] then
					self[Table.Stealing_player.Name .. "Died"]:Disconnect()
				end

				self[Table.Stealing_player.Name .. "Died"] = Table.Stealing_player.Character.Humanoid:GetPropertyChangedSignal("Health"):Connect(function()
					if Table.Stealing_player.Character.Humanoid.Health <= 0 then
						self[Table.Stealing_player.Name .. "Died"]:Disconnect()
						Stealed_Base.Give_back_stealed_brainrot()
					end
				end)
			end
		end

		self.Give_youtuber_in_hands(Table.Youtuber_model, Table.Stealing_player.Character)

		Add_stealed_youtuber_to_stealing_player()
		Remove_stealed_youtuber_from_origo_player()
	end
end

function Base_manager:Gates_opening(Boolean)
	if not self.Base_player then return end 
	if not Base_manager[self.Base_player] then return end

	local function Gates_opening_function()
		if Boolean then
			Base_manager[self.Base_player]["Gate_new"]:Open()
			Base_manager[self.Base_player]["Gate_new2"]:Open()
			pcall(function()
				Server_cesty.Replicated_storage.Eventy.RemoteFunction:InvokeClient(self.Base_player, {Type = "Show_gates", Base = self.Base})
				Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireAllClients({Type = "Gate_status_texts", Base = self.Base, Spin = false})
			end)
		elseif not Boolean then
			Base_manager[self.Base_player]["Gate_new"]:Close()
			Base_manager[self.Base_player]["Gate_new2"]:Close()
			local function Status_hidden_function()
				local Base_hidden
				repeat
					task.wait()
					Base_hidden = Server_cesty.Replicated_storage.Eventy.RemoteFunction:InvokeClient(self.Base_player, {Type = "Hide_gates", Base = self.Base})
				until Base_hidden
			end

			task.spawn(Status_hidden_function)

			Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireAllClients({Type = "Gate_status_texts", Base = self.Base, Spin = true})
		end
	end

	if not Base_manager[self.Base_player]["Gate_new"] then
		Base_manager[self.Base_player]["Gate_new"] = Gate_manager.New_gate(self.Base.Gates) 
		Base_manager[self.Base_player]["Gate_new2"] = Gate_manager.New_gate(self.Base["2floor"].Gates, {Y = 26.66, YC = 20.54}) 
		
		Gates_opening_function()
	else
		Gates_opening_function()
	end 
end

function Base_manager:LockBase()
	if self then
		local Base_player = self.Base_player
		if not Base_player then return end 
		if self.cant_player_control_lock then return end 
		local Base_player_data 
		repeat
			task.wait(.1)
			Base_player_data = Data_manager.GetData(Base_player)
		until Base_player_data

		local Base_player_lock_time = Base_player_data.Lock_time

		self:Gates_opening(false)
		self.cant_player_control_lock = true
		self.Base:SetAttribute("Locked", true)
		Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(self.Base_player, {Type = "Play_sound", Sound_name = "Success"})

		local function Set_message_all_clients_to_my_base(Message)
			Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireAllClients({Type = "Base_countdown_update", Message = Message, Base = self.Base})
		end

		local function Countdowning()
			while Base_player_lock_time > 0 do 
				Base_player_lock_time -= 1
				Set_message_all_clients_to_my_base(`{self.Base_countdown_text} {Base_player_lock_time}s`)
				task.wait(1)
			end
		end

		Set_message_all_clients_to_my_base(`{self.Base_countdown_text} {Base_player_lock_time}s`)
		self.countdown_task = task.spawn(Countdowning)

		task.delay(Base_player_lock_time, function()
			if self.countdown_task then
				task.cancel(self.countdown_task)
			end

			if not Base_player or not Base_manager[Base_player] then return end 

			self.cant_player_control_lock = false
			self.Base:SetAttribute("Locked", false)
			self:Gates_opening(true)

			Set_message_all_clients_to_my_base("")
			Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player,{Type = "Base_countdown_update", Message = "Lock base", Base = self.Base})
		end)
	end
end

function Base_manager:Load_base_one(Youtuber_info) -- Load one youtuber in player base
	local Base_player = self.Base_player
	local Base = self.Base 
	if Base and Base_player then
		local Player_data = Data_manager.GetData(Base_player)
		if not Player_data then return end 
		local Youtuber_infos = require(Server_cesty.Local_moduly.Parent.Moduly_info.Youtubers)
		local Youtuber_info_module = Youtuber_infos[Youtuber_info.Youtuber_name]
		
		local function Get_youtuber_data_index()
			for index, table  in pairs(Player_data.Youtubers) do
				if table.Youtuber_specialid == Youtuber_info.Youtuber_specialid then
					return index
				end
			end
		end
		
		local function Setup_collector(Sit_part)
			if not Sit_part.Parent:FindFirstChild("Collector") then
				local Collector = Instance.new("Part", Sit_part.Parent)
				Collector.Name = "Collector"
				Collector.Color = Color3.fromRGB(47, 218, 0)
				Collector.Position = Sit_part.Position + Sit_part.CFrame.LookVector * 8 - Vector3.new(0,2.5,0)
				Collector.Size = Vector3.new(3,1,3)
				--Collector.CFrame = CFrame.lookAt(Collector.Position, Sit_part.Position)
				Collector.CanCollide = false
				Collector.Anchored = true
				Collector.Transparency = 1
				
				local Collect_money = Server_cesty.Replicated_storage.Asety.Collect_money:Clone()
				Collect_money.Parent = Collector
				Collect_money.Enabled = false

				local Collect_money_textlabel = Collect_money.Frame.TextLabel

				local Player_youtuber_data = self.Find_youtuber_table_by_specialid({Youtuber_specialid = Youtuber_info.Youtuber_specialid, Data = Player_data})
		        
				
				local Can_collector_touch = true
				Collector.Touched:Connect(function(Hit)
					local Hitted_player = Server_cesty.Players:GetPlayerFromCharacter(Hit.Parent)
					if Hitted_player then
						if Hitted_player == Base_player then
							if Player_youtuber_data and Can_collector_touch then
								Can_collector_touch = false
								task.delay(1, function()
									Can_collector_touch = true
								end)
								local Local_youtuber_earnings = Get_youtuber_data_index()
								if not Local_youtuber_earnings then return end 
								
								Data_manager.Add_value(Base_player, "Coins", Player_youtuber_data.Earnings)
                                
								Player_data.Youtubers[Local_youtuber_earnings].Earnings = 0
								Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player, {Type = "Base_collector_update", Collector = Collector, Earnings = Player_data.Youtubers[Local_youtuber_earnings].Earnings, Additional_number = 0})
							end
						end
					end
				end)

				local Can_add_earnings = true
				if self[Collector] then
					self[Collector]:Disconnect()
				end
				
				self[Collector] = Server_cesty.Run_service.Stepped:Connect(function()
					if Can_add_earnings then
						Can_add_earnings = false
						task.delay(1, function()
							Can_add_earnings = true
						end)
						
						local Local_youtuber_earnings = Get_youtuber_data_index()
						if not Local_youtuber_earnings then return end 
						
						if Player_data.Youtubers[Local_youtuber_earnings] then
							if Player_data.Youtubers[Local_youtuber_earnings].Earnings then
								Player_data.Youtubers[Local_youtuber_earnings].Earnings += Youtuber_info_module.Earnings

								Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player, {Type = "Base_collector_update", Collector = Collector, Earnings = Player_data.Youtubers[Local_youtuber_earnings].Earnings})
							end
						end
					end
				end)
			end
		end
		
		local function Place_youtuber_in_place(Youtuber,Place, Special_id)
			if not Place then return end 
			local Place_sit_part = Place.Sit_part
			local Youtuber_sit_cframe = Place_sit_part.CFrame

			Place_sit_part:SetAttribute("Sitting_youtuber", Special_id)
			Youtuber:SetAttribute("Youtuber", true)
			Youtuber:PivotTo(Youtuber_sit_cframe)
			
			Setup_collector(Place_sit_part)
		end

		local function Get_youtuber(Youtuber_name)
			local Possible_youtuber = Server_cesty.Replicated_storage.Asety.Youtubers:FindFirstChild(Youtuber_name)
			if Possible_youtuber then
				local cloned = Possible_youtuber:Clone()
				return cloned
			end
		end

		local function Create_youtuber_proximity(Youtuber)
			local Proximity_prompt = Instance.new("ProximityPrompt", Youtuber.PrimaryPart)
			Proximity_prompt.Name = "Youtuber_proximity"
			Proximity_prompt.RequiresLineOfSight = false
			Proximity_prompt:SetAttribute("DescendantAdded_check", true)
			Proximity_prompt.Exclusivity = Enum.ProximityPromptExclusivity.AlwaysShow
			Proximity_prompt.MaxActivationDistance = 11
			return Proximity_prompt
		end
		
		
		local function Set_sitting_animation(Youtuber)
			local function Get_humanoid()
				for i, v in pairs(Youtuber:GetDescendants()) do 
					if v:IsA("Humanoid") then
						return v
					end
				end
			end
			Youtuber.PrimaryPart.Anchored = true
			local Youtuber_humanoid = Get_humanoid()
			local Animation = Instance.new("Animation", Youtuber)
			Animation.AnimationId = Info_animations.Sitting

			local Loaded_animation = Youtuber_humanoid.Animator:LoadAnimation(Animation)
			Loaded_animation:Play()
			
			local Youtubers_sit_settings = {
				["PewLivePie"] = CFrame.new(0,0.25,0), --* CFrame.Angles(0,math.rad(180),0),
				["DanTTM"] = CFrame.new(0,0.15,0),
				["Ninja"] = CFrame.new(0,-0.15,0),
				["SSSniperWolf"] = CFrame.new(0,0.15,0),
				["Kurtis Corner"] = CFrame.new(0,0.1,0),
				["Jack Pembook"] = CFrame.new(0,0.1,0),
				["Jake Train"] = CFrame.new(0,0.3,0),
				["Blipbi"] = CFrame.new(0,0.3,0),
			}
			local Additional_rotation = Youtubers_sit_settings[Youtuber.Name] or CFrame.Angles(0,0,0)
			Youtuber:PivotTo(Youtuber:GetPivot() * CFrame.new(0,1,0) * Additional_rotation)
		end

		local function Set_ui(Youtuber)
			local Youtuber_info_billboard = Server_cesty.Replicated_storage.Asety.Youtuber_info_billboard:Clone()
			Youtuber_info_billboard.Parent = Youtuber.PrimaryPart
			Youtuber_info_billboard.Name_text.Text = Youtuber.Name
			
			local Rarity_colors = require(Server_cesty.Local_moduly.Parent.Moduly_info.Rarities_colors)
			local Youtuber_info_rarity = Youtuber_info_module.Rarity
			local Youtuber_info_color = Rarity_colors[Youtuber_info_rarity]
			Youtuber_info_billboard.Rarity.Text = Youtuber_info_rarity
			Youtuber_info_billboard.Rarity.TextColor3 = Youtuber_info_color
			
			Youtuber_info_billboard.Per_s.Text = `${Youtuber_info_module.Earnings}/s`
			Youtuber_info_billboard.Price.Text = `${Youtuber_info_module.SellingPrice}`
		end
		
		if not Youtuber_info then warn("No youtuber table") return end

		local Youtuber_name = Youtuber_info.Youtuber_name
		local Youtuber_location = Youtuber_info.Youtuber_location
		local Youtuber_floor = Youtuber_info.Youtuber_floor
		local Youtuber_specialid = Youtuber_info.Youtuber_specialid
		if not Youtuber_floor then print(Youtuber_info, " dont have any floor set") return end
		if not Youtuber_location then print(Youtuber_info, "dont have any location") return end 
		
		local Youtuber_placing_floor = Base:FindFirstChild(Youtuber_floor)
		if Youtuber_placing_floor then
			local Youtuber_placing_place = Youtuber_placing_floor:FindFirstChild(Youtuber_location)
			if Youtuber_placing_place then
				local Youtuber = Get_youtuber(Youtuber_name)
				if Youtuber then
					Youtuber.Parent = Youtuber_placing_place
					Youtuber:SetAttribute("Youtuber_specialid", Youtuber_specialid)
					local Youtuber_selling_price = Youtuber_info_module.SellingPrice
					
					Place_youtuber_in_place(Youtuber, Youtuber_placing_place, Youtuber_specialid)
					Set_ui(Youtuber)
					
					local Youtuber_proximity_prompt = Create_youtuber_proximity(Youtuber)
					local function Setup_proximity()
						Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player, {Type = "Youtuber_proximity_sell", Base = Base})

						Youtuber_proximity_prompt.Triggered:Connect(function(Triggered_player)
							if Triggered_player == Base_player then
								
								Data_manager.Add_value(Base_player, "Coins", Youtuber_selling_price/2)
								self:Remove_youtuber({Youtuber_specialid = Youtuber_specialid, Base = Base, Youtuber_model = Youtuber})
							else
								self:Steal_youtuber({Youtuber_specialid = Youtuber_specialid, Stealing_player = Triggered_player, Youtuber_model = Youtuber})
							end
						end)

						for index, player_ in pairs(Server_cesty.Players:GetPlayers()) do 
							if player_ ~= Base_player then
								Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(player_, {Type = "Youtuber_proximity_steal", Base = Base})
							end
						end

						Server_cesty.Replicated_storage.Eventy.RemoteEvent:FireClient(Base_player, {Type = "Youtuber_proximity_move", Youtuber = Youtuber})
					end
					Setup_proximity()
					Set_sitting_animation(Youtuber)
				end
			end
		end
	end
end

function Base_manager:Load_base() -- Load every youtuber in player base
	local Base_player = self.Base_player
	local Base = self.Base 

	if Base and Base_player then
		local Player_data = Data_manager.GetData(Base_player)
		local Player_data_youtubers = Player_data.Youtubers
	
		self:Remove_npcs()
		for Index_number, Youtuber_info in pairs(Player_data_youtubers) do 
			if not Youtuber_info.Owner and not Youtuber_info.Moving then
				self:Load_base_one(Youtuber_info)
			end
		end
	end
end

return Base_manager
