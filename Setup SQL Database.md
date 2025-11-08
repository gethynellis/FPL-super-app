## Create SQL Server database

This is base datamodel for the FPL-Sup_Spp


```SQL

Create the following Power BI Semantic Model Tables into a series of SQL Statements that create the tables in SQL Server. The relationships should be maintained in SQL Server

---

createOrReplace

	table 'Club Data (DIM)'
		lineageTag: 204c6330-1657-45ac-960f-fc261f7d3025

		column 'Team Code'
			dataType: string
			lineageTag: 595c9c81-6909-4be4-a1cf-7f51e5fa8625
			summarizeBy: none
			sourceColumn: Team Code

			annotation SummarizationSetBy = Automatic

		column 'Team ID'
			dataType: string
			lineageTag: 64929ca0-cbc7-48c1-81b7-b2e5e7980979
			summarizeBy: none
			sourceColumn: Team ID

			annotation SummarizationSetBy = Automatic

		column 'Team name'
			dataType: string
			lineageTag: d2fb334c-3ca0-46d1-ab67-3129bb1e21f2
			summarizeBy: none
			sourceColumn: Team name

			annotation SummarizationSetBy = Automatic

			annotation __PBI_SemanticLinks = [{"LinkTarget":{"TableName":"Club Data (DIM)","TableItemName":"Team name (groups) 3","ObjectType":4},"LinkType":"UsedInGroup"},{"LinkTarget":{"TableName":"Club Data (DIM)","TableItemName":"Team name (groups)","ObjectType":4},"LinkType":"UsedInGroup"}]

		column pulse_id
			dataType: string
			lineageTag: 0740916a-7cfc-49f9-a712-1297fcf3b5dd
			summarizeBy: none
			sourceColumn: pulse_id

			annotation SummarizationSetBy = Automatic

		column 'Team name (groups) 3' =
				SWITCH(
					TRUE,
					ISBLANK('Club Data (DIM)'[Team name]),
					"(Blank)",
					'Club Data (DIM)'[Team name] IN {"Leicester",
						"Ipswich",
						"Southampton"},
					"Ipswich & Leicester & Southampton",
					"Other"
				)
			lineageTag: dad1b0ff-df90-42ee-aca9-eff8cd836a31
			summarizeBy: none

			extendedProperty GroupingMetadata = {"version":0,"groupedColumns":[{"Column":{"Expression":{"SourceRef":{"Entity":"Club Data (DIM)"}},"Property":"Team name"}}]}

			annotation GroupingDesignState = {"Version":0,"Sources":[{"Name":"c","Entity":"Club Data (DIM)"}],"GroupedColumns":[{"Column":{"Expression":{"SourceRef":{"Source":"c"}},"Property":"Team name"}}],"GroupItems":[{"DisplayName":"(Blank)","BlankDefaultPlaceholder":true},{"DisplayName":"Ipswich & Leicester & Southampton","Expression":{"In":{"Expressions":[{"Column":{"Expression":{"SourceRef":{"Source":"c"}},"Property":"Team name"}}],"Values":[[{"Literal":{"Value":"'Leicester'"}}],[{"Literal":{"Value":"'Ipswich'"}}],[{"Literal":{"Value":"'Southampton'"}}]]}}},{"DisplayName":"Other"}]}

			annotation SummarizationSetBy = Automatic

		column 'Team name (groups)' =
				SWITCH(
					TRUE,
					ISBLANK('Club Data (DIM)'[Team name]),
					"(Blank)",
					'Club Data (DIM)'[Team name] IN {"Liverpool",
						"Chelsea",
						"Bournemouth",
						"Man City"},
					"Bournemouth & Chelsea & Liverpool & Man City",
					"Other"
				)
			lineageTag: a994ef46-c3b3-47d1-9ae1-a1085537b99e
			summarizeBy: none

			extendedProperty GroupingMetadata = {"version":0,"groupedColumns":[{"Column":{"Expression":{"SourceRef":{"Entity":"Club Data (DIM)"}},"Property":"Team name"}}]}

			annotation GroupingDesignState = {"Version":0,"Sources":[{"Name":"c","Entity":"Club Data (DIM)"}],"GroupedColumns":[{"Column":{"Expression":{"SourceRef":{"Source":"c"}},"Property":"Team name"}}],"GroupItems":[{"DisplayName":"(Blank)","BlankDefaultPlaceholder":true},{"DisplayName":"Bournemouth & Chelsea & Liverpool & Man City","Expression":{"In":{"Expressions":[{"Column":{"Expression":{"SourceRef":{"Source":"c"}},"Property":"Team name"}}],"Values":[[{"Literal":{"Value":"'Liverpool'"}}],[{"Literal":{"Value":"'Chelsea'"}}],[{"Literal":{"Value":"'Bournemouth'"}}],[{"Literal":{"Value":"'Man City'"}}]]}}},{"DisplayName":"Other"}]}

			annotation SummarizationSetBy = Automatic

		partition 'Club Data (DIM)-a2e2192d-6983-418a-b6bf-84f9924af321' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    teams = Source[teams],
					    #"Converted to Table" = Table.FromList(teams, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"code", "draw", "form", "id", "loss", "name", "played", "points", "position", "short_name", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away", "pulse_id"}, {"code", "draw", "form", "id", "loss", "name", "played", "points", "position", "short_name", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away", "pulse_id"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Column1",{{"code", type text}}),
					    #"Removed Columns" = Table.RemoveColumns(#"Changed Type",{"draw", "form", "loss", "played", "points", "position", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away"}),
					    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{{"id", "Team ID"}, {"code", "Team Code"}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"Team ID", type text}}),
					    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"name", "Team name"}}),
					    #"Removed Columns1" = Table.RemoveColumns(#"Renamed Columns1",{"short_name"})
					in
					    #"Removed Columns1"

		annotation PBI_ResultType = Table

		annotation PBI_NavigationStepName = Navigation




---


createOrReplace

	table ClubAnalysis
		lineageTag: c256e28d-b454-45b4-a9bf-5574f00879b0

		column chance_of_playing_next_round
			dataType: string
			lineageTag: bb12536e-9d2d-4812-9cf0-b9268e59d976
			summarizeBy: none
			sourceColumn: chance_of_playing_next_round

			annotation SummarizationSetBy = Automatic

		column chance_of_playing_this_round
			dataType: string
			lineageTag: 35bd6fa2-f76d-4034-a607-23212f5ec9e4
			summarizeBy: none
			sourceColumn: chance_of_playing_this_round

			annotation SummarizationSetBy = Automatic

		column code
			dataType: string
			lineageTag: eaf66989-3a43-4798-bd38-3f4f1e5b4eac
			summarizeBy: none
			sourceColumn: code

			annotation SummarizationSetBy = Automatic

		column cost_change_event
			dataType: string
			lineageTag: bee3ce53-7a2a-46af-920d-10c38339f972
			summarizeBy: none
			sourceColumn: cost_change_event

			annotation SummarizationSetBy = Automatic

		column cost_change_event_fall
			dataType: string
			lineageTag: f159f4d0-706d-4941-a640-3a4eed31e36a
			summarizeBy: none
			sourceColumn: cost_change_event_fall

			annotation SummarizationSetBy = Automatic

		column cost_change_start
			dataType: string
			lineageTag: 6c4fdd75-0170-47ab-a9f5-521784f8a370
			summarizeBy: none
			sourceColumn: cost_change_start

			annotation SummarizationSetBy = Automatic

		column cost_change_start_fall
			dataType: string
			lineageTag: 5e6d357a-0ac6-41ca-899d-c15e81e44e3f
			summarizeBy: none
			sourceColumn: cost_change_start_fall

			annotation SummarizationSetBy = Automatic

		column dreamteam_count
			dataType: string
			lineageTag: 08240fcc-18d6-475e-9a55-f48fe29007aa
			summarizeBy: none
			sourceColumn: dreamteam_count

			annotation SummarizationSetBy = Automatic

		column 'Position ID'
			dataType: string
			lineageTag: ff744a23-f45c-451d-bda4-4e323eab48fb
			summarizeBy: none
			sourceColumn: Position ID

			annotation SummarizationSetBy = Automatic

		column ep_next
			dataType: double
			lineageTag: c64e2f0a-28c3-4273-b564-bc61ea764980
			summarizeBy: sum
			sourceColumn: ep_next

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column ep_this
			dataType: double
			lineageTag: 60d6da5f-794d-4182-896b-97f2db49aff6
			summarizeBy: sum
			sourceColumn: ep_this

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column event_points
			dataType: string
			lineageTag: 3df50818-7f81-47cb-b8d4-e917ef46c886
			summarizeBy: none
			sourceColumn: event_points

			annotation SummarizationSetBy = Automatic

		column first_name
			dataType: string
			lineageTag: 39213876-326e-41a5-9617-c0c37360480a
			summarizeBy: none
			sourceColumn: first_name

			annotation SummarizationSetBy = Automatic

		column form
			dataType: string
			lineageTag: 54936bfd-3249-49c5-83e1-561156f3e63d
			summarizeBy: none
			sourceColumn: form

			annotation SummarizationSetBy = Automatic

		column id
			dataType: string
			lineageTag: e9346240-7be7-49b7-9c78-dbc9f45ec1ef
			summarizeBy: none
			sourceColumn: id

			annotation SummarizationSetBy = Automatic

		column in_dreamteam
			dataType: string
			lineageTag: bf616a3c-d602-499d-b8a8-0bfcf5315cd5
			summarizeBy: none
			sourceColumn: in_dreamteam

			annotation SummarizationSetBy = Automatic

		column news
			dataType: string
			lineageTag: 42c55fd6-3e9a-4f05-a7c3-c96582f9ad13
			summarizeBy: none
			sourceColumn: news

			annotation SummarizationSetBy = Automatic

		column news_added
			dataType: string
			lineageTag: 2ddd8956-a8fd-4ebd-a00c-ce6888d6cef2
			summarizeBy: none
			sourceColumn: news_added

			annotation SummarizationSetBy = Automatic

		column now_cost
			dataType: string
			lineageTag: 2108e72c-3911-4b3a-9aa1-6cfd8e93a1dd
			summarizeBy: none
			sourceColumn: now_cost

			annotation SummarizationSetBy = Automatic

		column photo
			dataType: string
			lineageTag: 48d78f4b-8cdf-465f-af20-c6bfc250f9f6
			summarizeBy: none
			sourceColumn: photo

			annotation SummarizationSetBy = Automatic

		column points_per_game
			dataType: string
			lineageTag: 79487ea1-74ca-4355-992b-0f15001e66f6
			summarizeBy: none
			sourceColumn: points_per_game

			annotation SummarizationSetBy = Automatic

		column second_name
			dataType: string
			lineageTag: fc544c23-f356-4787-a528-ec62df5a17cd
			summarizeBy: none
			sourceColumn: second_name

			annotation SummarizationSetBy = Automatic

		column selected_by_percent
			dataType: string
			lineageTag: 62b18eab-8e8e-44a1-8f92-c5ff95574e2a
			summarizeBy: none
			sourceColumn: selected_by_percent

			annotation SummarizationSetBy = Automatic

		column special
			dataType: string
			lineageTag: 35435108-4d4b-47ea-87d4-ae05c975d557
			summarizeBy: none
			sourceColumn: special

			annotation SummarizationSetBy = Automatic

		column squad_number
			dataType: string
			lineageTag: 0b9976ad-ffb8-4e84-8a61-ca4f23e7a0b9
			summarizeBy: none
			sourceColumn: squad_number

			annotation SummarizationSetBy = Automatic

		column status
			dataType: string
			lineageTag: c6f51c74-49d3-4eae-9a6a-30a90a807d20
			summarizeBy: none
			sourceColumn: status

			annotation SummarizationSetBy = Automatic

		column team
			dataType: string
			lineageTag: 0191dd50-37ee-47ce-871c-f8ee09331c45
			summarizeBy: none
			sourceColumn: team

			annotation SummarizationSetBy = Automatic

		column team_code
			dataType: string
			lineageTag: 1580f867-8171-4ff7-a5a8-0aee3eefc98d
			summarizeBy: none
			sourceColumn: team_code

			annotation SummarizationSetBy = Automatic

		column total_points
			dataType: string
			lineageTag: 7b40704d-14ec-4c5c-8a7b-a4192f28e6c2
			summarizeBy: none
			sourceColumn: total_points

			annotation SummarizationSetBy = Automatic

		column transfers_in
			dataType: string
			lineageTag: 32dc24fe-a7e1-4705-9420-72ba100f51f1
			summarizeBy: none
			sourceColumn: transfers_in

			annotation SummarizationSetBy = Automatic

		column transfers_in_event
			dataType: string
			lineageTag: 04ced3e4-5a29-4ef5-a3a0-6d67f6c1bc7a
			summarizeBy: none
			sourceColumn: transfers_in_event

			annotation SummarizationSetBy = Automatic

		column transfers_out
			dataType: string
			lineageTag: 96ff3b86-b0f8-4543-8ba4-9ec9a24401ef
			summarizeBy: none
			sourceColumn: transfers_out

			annotation SummarizationSetBy = Automatic

		column transfers_out_event
			dataType: string
			lineageTag: abbf448d-68a4-4710-837b-953f971ecd99
			summarizeBy: none
			sourceColumn: transfers_out_event

			annotation SummarizationSetBy = Automatic

		column value_form
			dataType: string
			lineageTag: b017b241-3f29-4081-8cf3-b7bfd661148e
			summarizeBy: none
			sourceColumn: value_form

			annotation SummarizationSetBy = Automatic

		column value_season
			dataType: string
			lineageTag: 935ba17d-267c-4653-9095-c177f50ee6d2
			summarizeBy: none
			sourceColumn: value_season

			annotation SummarizationSetBy = Automatic

		column web_name
			dataType: string
			lineageTag: 715dada2-f7ec-4d1c-a24a-ec09a5d0f7f4
			summarizeBy: none
			sourceColumn: web_name

			annotation SummarizationSetBy = Automatic

		column minutes
			dataType: string
			lineageTag: b3c51667-3cbc-47e1-bed7-8dd330158d8b
			summarizeBy: none
			sourceColumn: minutes

			annotation SummarizationSetBy = Automatic

		column goals_scored
			dataType: string
			lineageTag: 7503f2a1-c941-44df-9f40-dc4bfcbb5f5a
			summarizeBy: none
			sourceColumn: goals_scored

			annotation SummarizationSetBy = Automatic

		column assists
			dataType: string
			lineageTag: ff1c2d95-b3f7-4917-86cc-c4701784319a
			summarizeBy: none
			sourceColumn: assists

			annotation SummarizationSetBy = Automatic

		column clean_sheets
			dataType: string
			lineageTag: c14bbde0-dd9d-41c3-b34f-7d92e0ce5227
			summarizeBy: none
			sourceColumn: clean_sheets

			annotation SummarizationSetBy = Automatic

		column goals_conceded
			dataType: string
			lineageTag: 7c8ddbad-392b-4df1-9f2c-61fba1689dc5
			summarizeBy: none
			sourceColumn: goals_conceded

			annotation SummarizationSetBy = Automatic

		column own_goals
			dataType: string
			lineageTag: f9b2778a-2fe6-4ce0-8e57-199ad4697927
			summarizeBy: none
			sourceColumn: own_goals

			annotation SummarizationSetBy = Automatic

		column penalties_saved
			dataType: string
			lineageTag: 4fb73dfe-a5ab-4480-8df1-3d22780625fc
			summarizeBy: none
			sourceColumn: penalties_saved

			annotation SummarizationSetBy = Automatic

		column penalties_missed
			dataType: string
			lineageTag: 323dca92-002c-48f7-a588-0f83b393a6e9
			summarizeBy: none
			sourceColumn: penalties_missed

			annotation SummarizationSetBy = Automatic

		column yellow_cards
			dataType: string
			lineageTag: 16bc3b54-a11b-445c-a90f-6bc5bdc900dc
			summarizeBy: none
			sourceColumn: yellow_cards

			annotation SummarizationSetBy = Automatic

		column red_cards
			dataType: string
			lineageTag: 4b4503f6-0cdf-4842-bb10-9e1f64f911cc
			summarizeBy: none
			sourceColumn: red_cards

			annotation SummarizationSetBy = Automatic

		column saves
			dataType: string
			lineageTag: 77242d75-b2fb-476b-ad21-413d24088dfa
			summarizeBy: none
			sourceColumn: saves

			annotation SummarizationSetBy = Automatic

		column bonus
			dataType: string
			lineageTag: 8038945e-bf91-4a47-a454-07277f4fd228
			summarizeBy: none
			sourceColumn: bonus

			annotation SummarizationSetBy = Automatic

		column bps
			dataType: string
			lineageTag: c4c8026d-ba26-4610-bdce-614ca7d29b0a
			summarizeBy: none
			sourceColumn: bps

			annotation SummarizationSetBy = Automatic

		column influence
			dataType: string
			lineageTag: 5b2085c3-201b-4f84-96bd-9a1da6467ffd
			summarizeBy: none
			sourceColumn: influence

			annotation SummarizationSetBy = Automatic

		column creativity
			dataType: string
			lineageTag: 13f95cc5-bb1d-4d61-b7d0-18ffddcbb46e
			summarizeBy: none
			sourceColumn: creativity

			annotation SummarizationSetBy = Automatic

		column threat
			dataType: string
			lineageTag: fad5d154-2e4f-4957-bbd8-70b46554d7e0
			summarizeBy: none
			sourceColumn: threat

			annotation SummarizationSetBy = Automatic

		column ict_index
			dataType: string
			lineageTag: c45acbe7-0e03-4b5e-8938-d57f34a63cd3
			summarizeBy: none
			sourceColumn: ict_index

			annotation SummarizationSetBy = Automatic

		column starts
			dataType: string
			lineageTag: 17ec4f9b-eaf8-4de7-b46e-91235aa0bb40
			summarizeBy: none
			sourceColumn: starts

			annotation SummarizationSetBy = Automatic

		column expected_goals
			dataType: string
			lineageTag: bbd35bf3-a1c9-4cb4-8da0-54178dfb320e
			summarizeBy: none
			sourceColumn: expected_goals

			annotation SummarizationSetBy = Automatic

		column expected_assists
			dataType: string
			lineageTag: 67c6bcf3-06a5-4eed-ad45-a33b90564fc6
			summarizeBy: none
			sourceColumn: expected_assists

			annotation SummarizationSetBy = Automatic

		column expected_goal_involvements
			dataType: string
			lineageTag: dd315fc6-1369-411e-9383-49af13edefc7
			summarizeBy: none
			sourceColumn: expected_goal_involvements

			annotation SummarizationSetBy = Automatic

		column expected_goals_conceded
			dataType: string
			lineageTag: 30d084b6-8ec1-44d1-811d-d010c480640f
			summarizeBy: none
			sourceColumn: expected_goals_conceded

			annotation SummarizationSetBy = Automatic

		column influence_rank
			dataType: string
			lineageTag: a76b1a1d-4d3f-4519-a15b-1d7001d0c08e
			summarizeBy: none
			sourceColumn: influence_rank

			annotation SummarizationSetBy = Automatic

		column influence_rank_type
			dataType: string
			lineageTag: 50e6c794-96d2-416f-847a-6314e136b1f6
			summarizeBy: none
			sourceColumn: influence_rank_type

			annotation SummarizationSetBy = Automatic

		column creativity_rank
			dataType: string
			lineageTag: cc857efb-d012-47cd-a56d-ea647ed0a485
			summarizeBy: none
			sourceColumn: creativity_rank

			annotation SummarizationSetBy = Automatic

		column creativity_rank_type
			dataType: string
			lineageTag: 8d423f65-74f7-4832-b9b2-b4a2f9b30c50
			summarizeBy: none
			sourceColumn: creativity_rank_type

			annotation SummarizationSetBy = Automatic

		column threat_rank
			dataType: string
			lineageTag: 5423a0d4-1e9e-4ec4-8539-4de2886740b7
			summarizeBy: none
			sourceColumn: threat_rank

			annotation SummarizationSetBy = Automatic

		column threat_rank_type
			dataType: string
			lineageTag: e75e62d4-da08-4060-95af-069c6f1ad986
			summarizeBy: none
			sourceColumn: threat_rank_type

			annotation SummarizationSetBy = Automatic

		column ict_index_rank
			dataType: string
			lineageTag: 0d5a7bbb-57ef-4389-b0cd-4b2a33cc2cdc
			summarizeBy: none
			sourceColumn: ict_index_rank

			annotation SummarizationSetBy = Automatic

		column ict_index_rank_type
			dataType: string
			lineageTag: 88085c47-c421-497f-91ea-5b6e5dd659ec
			summarizeBy: none
			sourceColumn: ict_index_rank_type

			annotation SummarizationSetBy = Automatic

		column corners_and_indirect_freekicks_order
			dataType: string
			lineageTag: 47ccab49-835f-462b-88b9-bbbe7f83e1e1
			summarizeBy: none
			sourceColumn: corners_and_indirect_freekicks_order

			annotation SummarizationSetBy = Automatic

		column corners_and_indirect_freekicks_text
			dataType: string
			lineageTag: e226ab11-6385-46db-b766-415828d1602e
			summarizeBy: none
			sourceColumn: corners_and_indirect_freekicks_text

			annotation SummarizationSetBy = Automatic

		column direct_freekicks_order
			dataType: string
			lineageTag: 6278517f-c1bc-4ed2-8a70-c30bfdcc04f8
			summarizeBy: none
			sourceColumn: direct_freekicks_order

			annotation SummarizationSetBy = Automatic

		column direct_freekicks_text
			dataType: string
			lineageTag: c22507d7-e511-4e4e-ab6a-19a0490b5807
			summarizeBy: none
			sourceColumn: direct_freekicks_text

			annotation SummarizationSetBy = Automatic

		column penalties_order
			dataType: string
			lineageTag: a2403f0a-74cd-48e6-aecc-d14a124bc6d2
			summarizeBy: none
			sourceColumn: penalties_order

			annotation SummarizationSetBy = Automatic

		column penalties_text
			dataType: string
			lineageTag: 7b0f1e27-9ba5-496b-9984-8339a9fd8685
			summarizeBy: none
			sourceColumn: penalties_text

			annotation SummarizationSetBy = Automatic

		column expected_goals_per_90
			dataType: string
			lineageTag: 497c24dc-0f53-4656-8de9-661a3935f604
			summarizeBy: none
			sourceColumn: expected_goals_per_90

			annotation SummarizationSetBy = Automatic

		column saves_per_90
			dataType: string
			lineageTag: dea73c4d-3b4e-48bb-aadb-baaf18360ea6
			summarizeBy: none
			sourceColumn: saves_per_90

			annotation SummarizationSetBy = Automatic

		column expected_assists_per_90
			dataType: string
			lineageTag: b9a745f4-b2cf-4fe2-9d45-6684f280c687
			summarizeBy: none
			sourceColumn: expected_assists_per_90

			annotation SummarizationSetBy = Automatic

		column expected_goal_involvements_per_90
			dataType: string
			lineageTag: 8b830885-e007-4c47-b3c7-59dbc53b2039
			summarizeBy: none
			sourceColumn: expected_goal_involvements_per_90

			annotation SummarizationSetBy = Automatic

		column expected_goals_conceded_per_90
			dataType: string
			lineageTag: b20c6ddd-44e4-4a8c-b874-1bcd734ae0d6
			summarizeBy: none
			sourceColumn: expected_goals_conceded_per_90

			annotation SummarizationSetBy = Automatic

		column goals_conceded_per_90
			dataType: string
			lineageTag: 16631099-5264-4968-a214-ce1579c7b5b7
			summarizeBy: none
			sourceColumn: goals_conceded_per_90

			annotation SummarizationSetBy = Automatic

		column now_cost_rank
			dataType: string
			lineageTag: 56e33263-e7da-4c40-81ec-d9f76407941c
			summarizeBy: none
			sourceColumn: now_cost_rank

			annotation SummarizationSetBy = Automatic

		column now_cost_rank_type
			dataType: string
			lineageTag: 367f0a26-883c-4c91-bdc2-d5171ad85ecf
			summarizeBy: none
			sourceColumn: now_cost_rank_type

			annotation SummarizationSetBy = Automatic

		column form_rank
			dataType: string
			lineageTag: fa1ab877-7f72-4b6c-b64f-f9f55570b14b
			summarizeBy: none
			sourceColumn: form_rank

			annotation SummarizationSetBy = Automatic

		column form_rank_type
			dataType: string
			lineageTag: 09abed9f-384a-4525-98f9-fe4cd585a649
			summarizeBy: none
			sourceColumn: form_rank_type

			annotation SummarizationSetBy = Automatic

		column points_per_game_rank
			dataType: string
			lineageTag: 023b9f75-87a0-403b-98e8-0986ad57790c
			summarizeBy: none
			sourceColumn: points_per_game_rank

			annotation SummarizationSetBy = Automatic

		column points_per_game_rank_type
			dataType: string
			lineageTag: 2d5b809c-69a5-4622-86af-2078b3a85c56
			summarizeBy: none
			sourceColumn: points_per_game_rank_type

			annotation SummarizationSetBy = Automatic

		column selected_rank
			dataType: string
			lineageTag: dd665fcf-d943-4f72-8928-070f75974c12
			summarizeBy: none
			sourceColumn: selected_rank

			annotation SummarizationSetBy = Automatic

		column selected_rank_type
			dataType: string
			lineageTag: 2184b960-2cb2-4c8d-828e-2a78c55d0636
			summarizeBy: none
			sourceColumn: selected_rank_type

			annotation SummarizationSetBy = Automatic

		column starts_per_90
			dataType: string
			lineageTag: 27057904-12f8-4300-96d2-6c717785eca5
			summarizeBy: none
			sourceColumn: starts_per_90

			annotation SummarizationSetBy = Automatic

		column clean_sheets_per_90
			dataType: string
			lineageTag: 571c6048-bbc7-4d9b-bfcf-88d8c399c01c
			summarizeBy: none
			sourceColumn: clean_sheets_per_90

			annotation SummarizationSetBy = Automatic

		partition ClubAnalysis = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    elements = Source[elements],
					    #"Converted to Table" = Table.FromList(elements, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"chance_of_playing_next_round", "chance_of_playing_this_round", "code", "cost_change_event", "cost_change_event_fall", "cost_change_start", "cost_change_start_fall", "dreamteam_count", "element_type", "ep_next", "ep_this", "event_points", "first_name", "form", "id", "in_dreamteam", "news", "news_added", "now_cost", "photo", "points_per_game", "second_name", "selected_by_percent", "special", "squad_number", "status", "team", "team_code", "total_points", "transfers_in", "transfers_in_event", "transfers_out", "transfers_out_event", "value_form", "value_season", "web_name", "minutes", "goals_scored", "assists", "clean_sheets", "goals_conceded", "own_goals", "penalties_saved", "penalties_missed", "yellow_cards", "red_cards", "saves", "bonus", "bps", "influence", "creativity", "threat", "ict_index", "starts", "expected_goals", "expected_assists", "expected_goal_involvements", "expected_goals_conceded", "influence_rank", "influence_rank_type", "creativity_rank", "creativity_rank_type", "threat_rank", "threat_rank_type", "ict_index_rank", "ict_index_rank_type", "corners_and_indirect_freekicks_order", "corners_and_indirect_freekicks_text", "direct_freekicks_order", "direct_freekicks_text", "penalties_order", "penalties_text", "expected_goals_per_90", "saves_per_90", "expected_assists_per_90", "expected_goal_involvements_per_90", "expected_goals_conceded_per_90", "goals_conceded_per_90", "now_cost_rank", "now_cost_rank_type", "form_rank", "form_rank_type", "points_per_game_rank", "points_per_game_rank_type", "selected_rank", "selected_rank_type", "starts_per_90", "clean_sheets_per_90"}, {"chance_of_playing_next_round", "chance_of_playing_this_round", "code", "cost_change_event", "cost_change_event_fall", "cost_change_start", "cost_change_start_fall", "dreamteam_count", "element_type", "ep_next", "ep_this", "event_points", "first_name", "form", "id", "in_dreamteam", "news", "news_added", "now_cost", "photo", "points_per_game", "second_name", "selected_by_percent", "special", "squad_number", "status", "team", "team_code", "total_points", "transfers_in", "transfers_in_event", "transfers_out", "transfers_out_event", "value_form", "value_season", "web_name", "minutes", "goals_scored", "assists", "clean_sheets", "goals_conceded", "own_goals", "penalties_saved", "penalties_missed", "yellow_cards", "red_cards", "saves", "bonus", "bps", "influence", "creativity", "threat", "ict_index", "starts", "expected_goals", "expected_assists", "expected_goal_involvements", "expected_goals_conceded", "influence_rank", "influence_rank_type", "creativity_rank", "creativity_rank_type", "threat_rank", "threat_rank_type", "ict_index_rank", "ict_index_rank_type", "corners_and_indirect_freekicks_order", "corners_and_indirect_freekicks_text", "direct_freekicks_order", "direct_freekicks_text", "penalties_order", "penalties_text", "expected_goals_per_90", "saves_per_90", "expected_assists_per_90", "expected_goal_involvements_per_90", "expected_goals_conceded_per_90", "goals_conceded_per_90", "now_cost_rank", "now_cost_rank_type", "form_rank", "form_rank_type", "points_per_game_rank", "points_per_game_rank_type", "selected_rank", "selected_rank_type", "starts_per_90", "clean_sheets_per_90"}),
					    #"Renamed Columns" = Table.RenameColumns(#"Expanded Column1",{{"element_type", "Position ID"}}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"Position ID", type text}}),
					    #"Sorted Rows" = Table.Sort(#"Changed Type",{{"expected_goals", Order.Descending}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Sorted Rows",{{"ep_this", type number}, {"ep_next", type number}})
					in
					    #"Changed Type1"

		annotation PBI_NavigationStepName = Navigation

		annotation PBI_ResultType = Table




---


createOrReplace

	table ClubAnalysisDim
		lineageTag: c715b6eb-92b0-4bf7-8c20-a71807d22432

		column 'Team Code'
			dataType: string
			lineageTag: df63e88e-a75d-492e-9b58-dd5dd70a2f5f
			summarizeBy: none
			sourceColumn: Team Code

			annotation SummarizationSetBy = Automatic

		column 'Team ID'
			dataType: string
			lineageTag: 01174022-f966-4810-9825-07aa777b32d0
			summarizeBy: none
			sourceColumn: Team ID

			annotation SummarizationSetBy = Automatic

		column 'Team name'
			dataType: string
			lineageTag: 02179410-9de3-4b0b-af4c-f2fa3d4bb161
			summarizeBy: none
			sourceColumn: Team name

			annotation SummarizationSetBy = Automatic

		column pulse_id
			dataType: string
			lineageTag: eb74a716-17c9-4013-8af6-745f2f88327a
			summarizeBy: none
			sourceColumn: pulse_id

			annotation SummarizationSetBy = Automatic

		partition ClubAnalysisDim = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    teams = Source[teams],
					    #"Converted to Table" = Table.FromList(teams, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"code", "draw", "form", "id", "loss", "name", "played", "points", "position", "short_name", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away", "pulse_id"}, {"code", "draw", "form", "id", "loss", "name", "played", "points", "position", "short_name", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away", "pulse_id"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Column1",{{"code", type text}}),
					    #"Removed Columns" = Table.RemoveColumns(#"Changed Type",{"draw", "form", "loss", "played", "points", "position", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away"}),
					    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{{"id", "Team ID"}, {"code", "Team Code"}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"Team ID", type text}}),
					    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"name", "Team name"}}),
					    #"Removed Columns1" = Table.RemoveColumns(#"Renamed Columns1",{"short_name"})
					in
					    #"Removed Columns1"

		annotation PBI_NavigationStepName = Navigation

		annotation PBI_ResultType = Table




---

createOrReplace

	table Date
		lineageTag: aee63ede-76b6-4773-b170-1e24f136a721

		column Date
			formatString: General Date
			lineageTag: dc369623-d275-42bc-bf7c-9c9ceb81177f
			summarizeBy: none
			isNameInferred
			sourceColumn: [Date]

			variation Variation
				isDefault
				relationship: 8c93e821-aa88-4e66-aaa1-abd10ac6eb76
				defaultHierarchy: LocalDateTable_07a247a4-f0ba-407b-bafa-8557a65ac9f5.'Date Hierarchy'

			annotation SummarizationSetBy = Automatic

		partition Date = calculated
			mode: import
			source = CALENDARAUTO()

		annotation PBI_Id = af8be72b903e4785b01643795333d328



---

createOrReplace

	table DateDim
		lineageTag: bce35e2e-d3d1-44d5-84cb-0a8c1d1721ca
		dataCategory: Time

		column Date
			dataType: dateTime
			isKey
			formatString: Long Date
			lineageTag: 7ee16430-1b24-47b3-a4c3-e3f7432a1893
			summarizeBy: none
			sourceColumn: Date

			annotation SummarizationSetBy = Automatic

			annotation UnderlyingDateTimeDataType = Date

		column DateTime
			dataType: dateTime
			formatString: General Date
			lineageTag: ba3697c5-cb00-4dfb-85a7-053b05963503
			summarizeBy: none
			sourceColumn: DateTime

			annotation SummarizationSetBy = Automatic

		column Year = "CY" & YEAR('DateDim'[Date])
			lineageTag: b120ca57-171c-4860-a317-7f7ccabf5f8f
			summarizeBy: none

			annotation SummarizationSetBy = Automatic

		column Month = FORMAT('DateDim'[Date], "YYYY-MM")
			lineageTag: 4a2addba-49fe-49c1-a43d-511acd708781
			summarizeBy: none
			sortByColumn: 'Month Order'

			changedProperty = SortByColumn

			annotation SummarizationSetBy = Automatic

		column 'Month Number' = MONTH([Date])
			formatString: 0
			lineageTag: 36fd06e0-b0f7-41ad-8d43-017bc443cc2b
			summarizeBy: none

			annotation SummarizationSetBy = Automatic

		column 'Month Order' = Year('DateDim'[Date])*100 + MONTH('DateDim'[Date])
			formatString: 0
			lineageTag: 12592962-ab8e-466d-9efe-ef6e928fe26c
			summarizeBy: none

			annotation SummarizationSetBy = Automatic

		column GameWeekNumber
			dataType: string
			lineageTag: 88293c64-aba5-4784-b7db-9561de2b6ed4
			summarizeBy: none
			sourceColumn: GameWeekNumber

			annotation SummarizationSetBy = Automatic

		hierarchy 'Month Hierarchy'
			lineageTag: 1c85ad8d-ea76-48b9-8e4b-fcf1dccd36ed

			level Month
				lineageTag: 113c78b2-9446-4ff8-8bb0-5c4b220cceed
				column: Month

		partition DateDim = m
			mode: import
			source = ```
					let
					    Source = {Number.From(#date(2023,8,1))..Number.From(#date(2025,6,30))},
					    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Changed Type" = Table.TransformColumnTypes(#"Converted to Table",{{"Column1", type date}}),
					    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"Column1", "Dim"}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"Dim", type date}}),
					    #"Duplicated Column" = Table.DuplicateColumn(#"Changed Type1", "Dim", "Dim - Copy"),
					    #"Renamed Columns1" = Table.RenameColumns(#"Duplicated Column",{{"Dim", "Date"}, {"Dim - Copy", "DateTime"}}),
					    #"Changed Type2" = Table.TransformColumnTypes(#"Renamed Columns1",{{"DateTime", type datetime}}),
					    #"Added Custom1" = Table.AddColumn(#"Changed Type2", "Custom", each 
					    let 
					        FilteredGW = Table.SelectRows(GWDim, (row) => row[GW Deadline] > _[DateTime])
					    in 
					        if Table.IsEmpty(FilteredGW) 
					        then List.Max(GWDim[GW ID]) 
					        else List.Min(FilteredGW[GW ID])
					),
					    #"Filtered Rows" = Table.SelectRows(#"Added Custom1", each true),
					    #"Renamed Columns3" = Table.RenameColumns(#"Filtered Rows",{{"Custom", "GameWeekNumber"}}),
					    #"Filtered Rows1" = Table.SelectRows(#"Renamed Columns3", each true)
					in
					    #"Filtered Rows1"
					```

		annotation PBI_ResultType = Table

		annotation PBI_NavigationStepName = Navigation

---

createOrReplace

	table 'Detailed Player Data (Fact)'
		lineageTag: b1bac7db-f51d-4422-9751-513539dbafa4

		measure 'Team Ranks' =
				RANKX(
				ALL('Club Data (DIM)'[Team name])
				,[Total Player Points])
			formatString: 0
			lineageTag: 8fd45f38-8460-4101-85b7-241b76e95d3d

		measure 'Total Player Points' = CALCULATE(SUM('Detailed Player Data (Fact)'[Total Points]))
			formatString: 0
			lineageTag: 5846c8cb-93d9-4d49-bc49-b2a753a4064b

		measure 'Average Points' = AVERAGE('Detailed Player Data (Fact)'[Total Points])
			lineageTag: 152438b7-0594-4509-9061-edaa706aa8be

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure xGDiff = SUM('Detailed Player Data (Fact)'[Goals Scored]) - SUM('Detailed Player Data (Fact)'[expected_goals])
			lineageTag: d3805302-fc96-4b6c-82aa-a3d07312f40f

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure xGDiff% = SUM('Detailed Player Data (Fact)'[Goals Scored]) / SUM('Detailed Player Data (Fact)'[expected_goals])
			formatString: 0.00%;-0.00%;0.00%
			lineageTag: 17d39158-d18d-428e-b8d0-2543e6782d88

		measure 'Total Own Goals Benefited' = ```
				
				SUMX(
				    VALUES('Club Data (DIM)'[Team ID]), 
				    CALCULATE(
				        SUM('Detailed Player Data (Fact)'[Own goals]),
				        TREATAS(VALUES('Club Data (DIM)'[Team ID]), 'Detailed Player Data (Fact)'[Opposition Team ID])
				    )
				)
				
				```
			formatString: 0
			lineageTag: f2b23a71-378d-43b0-9786-6376949ded68

		measure 'Total Point Previous Month' = CALCULATE(SUM('Detailed Player Data (Fact)'[Total Points]), PREVIOUSMONTH(DateDim[Date]))
			formatString: 0
			lineageTag: c616961b-ab65-4ffa-a502-fecd7633e784

		measure Form =
				
				VAR TodayDate = MAX('Detailed Player Data (Fact)'[Kickoff_Time])
				VAR StartDate = TodayDate - 30
				
				RETURN
				CALCULATE(
				    AVERAGE('Detailed Player Data (Fact)'[Total Points]),
				    'Detailed Player Data (Fact)'[kickoff_time] >= StartDate &&
				    'Detailed Player Data (Fact)'[kickoff_time] <= TodayDate
				)
			lineageTag: 83694f45-0ba1-4cb1-980b-d903c1b686b1

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'Latest Player Price' = ```
				
				VAR CurrentGameweekID = 
				    SELECTEDVALUE('GWDim'[GW ID])
				    
				RETURN
				    CALCULATE(
				        MAX('Detailed Player Data (Fact)'[Price]), 
				        GWDim[is_current] = TRUE
				    )
				
				```
			formatString: 0
			lineageTag: 1872d044-981e-48ab-8ecb-4f7fa0bb4d23

		measure AvgMinutesPlayed = ```
				
				VAR PlayersWithMinutes =
				    FILTER ( 'Detailed Player Data (Fact)', 'Detailed Player Data (Fact)'[Minutes played] > 0 )
				RETURN
				    AVERAGEX ( PlayersWithMinutes, 'Detailed Player Data (Fact)'[Minutes played] )
				
				```
			lineageTag: 78581165-4fca-4f91-8a84-0bd50ae86e9c

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure Measure
			lineageTag: 9c5dea7f-34dd-4d94-b9e0-cdc6bc7835a0

			annotation 43dbc3e8-3a1c-4b6f-9923-b49ff7d6691c = True

		measure AvgMinutesPerClub = ```
				
				VAR TotalMinutesPlayed =
				    SUM ( 'Detailed Player Data (Fact)'[Minutes played] )
				
				VAR PlayersWithMinutes =
				    CALCULATE (
				        DISTINCTCOUNT ( 'Detailed Player Data (Fact)'[Player ID] ),
				        'Detailed Player Data (Fact)'[Minutes played] > 0
				    )
				
				RETURN
				    DIVIDE ( TotalMinutesPlayed, PlayersWithMinutes, 0 )
				
				```
			lineageTag: 6cee48ab-14f3-4c4d-a11f-a581bfbe4617

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'Clean sheets average per Player Name' =
				
				AVERAGEX(
					KEEPFILTERS(VALUES('Player Data (Dim)'[Player Name])),
					CALCULATE(SUM('Detailed Player Data (Fact)'[Clean sheets]))
				)
			formatString: 0
			lineageTag: 963d35ea-a334-4637-964a-911113b12c0e

			extendedProperty MeasureTemplate = {"version":0,"daxTemplateName":"AveragePerCategory"}

			changedProperty = FormatString

		measure 'No of Players' = DISTINCTCOUNT('Detailed Player Data (Fact)'[Player ID])
			formatString: 0
			lineageTag: 9cf0d6eb-2fba-4c34-8b2c-ed260844f98d

		measure 'Form Sum' = ```
				
				VAR TodayDate = MAX( 'Detailed Player Data (Fact)'[Kickoff_Time] )
				VAR StartDate = TodayDate - 30
				RETURN
				    CALCULATE(
				        SUM( 'Detailed Player Data (Fact)'[Total Points] ),
				        'Detailed Player Data (Fact)'[Kickoff_Time] >= StartDate,
				        'Detailed Player Data (Fact)'[Kickoff_Time] <= TodayDate
				    )
				
				```
			formatString: 0
			lineageTag: 4bf1f806-992b-44d4-8e99-f4b14831d7e9

		measure 'Avg Save Per Game' = AVERAGE('Detailed Player Data (Fact)'[saves])
			lineageTag: 0f11bc39-24ad-44fc-8deb-dcd239b9467b

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'Avg Goals Conceded' = AVERAGE('Detailed Player Data (Fact)'[Goals conceded])
			lineageTag: bc4c6ed0-921e-4d93-a4e8-33360019250a

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'Avg xgConceded per Game' = AVERAGE('Detailed Player Data (Fact)'[expected_goals_conceded])
			lineageTag: 5c05d94d-6f2c-4ac5-8b93-5b8ef82bb3f4

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'GK Expected Goals Conceded' = ```
				
				CALCULATE (
				    SUM ( 'Detailed Player Data (Fact)'[expected_goals_conceded] ),
				    'Position (DIM)'[Position Name] = "Goalkeepers"
				)
				
				```
			lineageTag: c231a9c3-622b-476e-a654-b18c7c78049a

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'GK Goals Conceded' = ```
				
				CALCULATE (
				    SUM ( 'Detailed Player Data (Fact)'[Goals conceded] ),
				    TREATAS ( {"Goalkeepers"}, 'Position (DIM)'[Position Name] )
				)
				
				```
			formatString: 0
			lineageTag: 5b384dda-4a85-4b2f-abaf-2e64e5dfbb9c

		measure 'Total Club Goals' = ```
				[Total Own Goals Benefited] + 
				```
			lineageTag: 67b2ac44-ac47-4f88-94f5-6ecf396409ea

		measure 'Goals Scored (Non Own Goals)' = SUM('Detailed Player Data (Fact)'[Goals Scored])
			formatString: 0
			lineageTag: 62d53127-fa49-43e1-8f62-1d20fefbad58

		measure 'Total Goals Scored' = [Goals Scored (Non Own Goals)] + [Total Own Goals Benefited]
			formatString: 0
			lineageTag: 2783b657-9404-4d93-af71-69a797e47a36

		measure 'mGoal Involvements' = SUM('Detailed Player Data (Fact)'[Goals Scored]) + SUM('Detailed Player Data (Fact)'[Assists made])
			formatString: 0
			lineageTag: 6996e063-537e-446a-b9fa-b5bccb864f2e

		measure 'mPlayer Rank by Goal Involvements' =
				
				RANKX (
				    ALL ('Player Data (Dim)'[Player Name] ),      -- remove filters so all players are ranked
				    [mGoal Involvements],             -- measure you already created
				    ,
				    DESC,                             -- rank highest first
				    DENSE                             -- ties get same rank, next rank is sequential
				)
			formatString: 0
			lineageTag: 11571c73-9f96-4b90-a840-8b7e6903c996

		measure 'Player Performance' = ([mGoal Involvements] - SUM('Detailed Player Data (Fact)'[expected_goal_involvements]))
			lineageTag: 3be0f336-856d-4954-9b07-0aa5bf2d53b7

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'Club Performance' = SUM('Detailed Player Data (Fact)'[Goals Scored]) - SUM('Detailed Player Data (Fact)'[expected_goals])
			lineageTag: 527b9c7d-6684-4aed-b568-386a06680b8c

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		measure 'Defensive Contributions > 10' = ```
				
				COUNTROWS (
				    FILTER (
				        'Detailed Player Data (Fact)',
				        'Detailed Player Data (Fact)'[defensive_contribution] >= 10
				    )
				)
				
				```
			formatString: 0
			lineageTag: 675c9eb0-b23d-49f6-8e52-0fece9b7a515

		measure 'Row Count' = ```
				
				COUNTROWS ( 'Detailed Player Data (Fact)' )
				
				```
			formatString: 0
			lineageTag: 9a2f1acf-7b90-4a38-b3b7-f5137854d592

		measure '% of games with FULL DC points Defenders' = [Defensive Contributions > 10] / [Row Count]
			formatString: 0.00%;-0.00%;0.00%
			lineageTag: 95112772-9fae-41ba-8a65-6a12826e32b3

		column Player
			dataType: int64
			formatString: 0
			lineageTag: e6282795-eead-4ede-b41f-8fcc9db73eee
			summarizeBy: sum
			sourceColumn: Player

			annotation SummarizationSetBy = Automatic

		column 'Fixture ID'
			dataType: string
			lineageTag: a997a600-2bba-4175-943b-cf4ec85f6a78
			summarizeBy: none
			sourceColumn: Fixture ID

			annotation SummarizationSetBy = Automatic

		column 'Opposition Team ID'
			dataType: string
			lineageTag: 920ee60d-d9ee-4ce3-98af-14314ef7a014
			summarizeBy: none
			sourceColumn: Opposition Team ID

			annotation SummarizationSetBy = Automatic

		column 'Total Points'
			dataType: int64
			formatString: 0
			lineageTag: 3bb9b26b-bb95-48c2-937f-4b0701f8c27f
			summarizeBy: sum
			sourceColumn: Total Points

			annotation SummarizationSetBy = User

		column was_home
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: 9edf2786-bbd9-4f46-b3b0-1e66897a6e0e
			summarizeBy: none
			sourceColumn: was_home

			annotation SummarizationSetBy = Automatic

		column kickoff_time
			dataType: dateTime
			formatString: General Date
			lineageTag: 122d144c-55d7-4a31-9d21-000d89dc028c
			summarizeBy: none
			sourceColumn: kickoff_time

			variation Variation
				isDefault
				relationship: 2d368d3a-1f54-420b-858d-a0b67d487698
				defaultHierarchy: LocalDateTable_f2a9968d-b940-46d7-861c-391fad084c1d.'Date Hierarchy'

			annotation SummarizationSetBy = Automatic

		column 'Home Teams Score'
			dataType: int64
			formatString: 0
			lineageTag: 8e82a2e6-efa1-4ddd-9e16-d37f3545ecb1
			summarizeBy: sum
			sourceColumn: Home Teams Score

			annotation SummarizationSetBy = Automatic

		column 'Away Team Score'
			dataType: int64
			formatString: 0
			lineageTag: cd831b05-71b9-47ce-9b55-b675002a9228
			summarizeBy: sum
			sourceColumn: Away Team Score

			annotation SummarizationSetBy = Automatic

		column 'Gameweek ID'
			dataType: string
			lineageTag: 96f6f930-dc24-4fdc-80ab-7cc130581ccc
			summarizeBy: none
			sourceColumn: Gameweek ID

			annotation SummarizationSetBy = Automatic

		column 'Minutes played'
			dataType: int64
			formatString: 0
			lineageTag: 836816f3-3cee-43b8-9cce-6f7cf9352b77
			summarizeBy: sum
			sourceColumn: Minutes played

			annotation SummarizationSetBy = Automatic

		column 'Goals Scored'
			dataType: int64
			formatString: 0
			lineageTag: b266812f-9fe3-4d5a-89aa-26b4189f4ab4
			summarizeBy: sum
			sourceColumn: Goals Scored

			annotation SummarizationSetBy = Automatic

		column 'Assists made'
			dataType: int64
			formatString: 0
			lineageTag: b5b641f4-9c6a-44d1-a015-f12cdb41f2e9
			summarizeBy: sum
			sourceColumn: Assists made

			annotation SummarizationSetBy = Automatic

		column 'Clean sheets'
			dataType: int64
			formatString: 0
			lineageTag: 75b44a31-14bf-483f-b79d-645f39760e8e
			summarizeBy: sum
			sourceColumn: Clean sheets

			annotation SummarizationSetBy = Automatic

		column 'Goals conceded'
			dataType: int64
			formatString: 0
			lineageTag: 14b65607-a673-406b-a7bf-7c3478300639
			summarizeBy: sum
			sourceColumn: Goals conceded

			annotation SummarizationSetBy = Automatic

		column 'Own goals'
			dataType: int64
			formatString: 0
			lineageTag: c0ec7ef8-cacf-4a91-a2f9-e8d7b0fb871e
			summarizeBy: sum
			sourceColumn: Own goals

			annotation SummarizationSetBy = Automatic

		column penalties_saved
			dataType: int64
			formatString: 0
			lineageTag: d17ac6a5-bcf3-440a-83f3-c1a1161dca96
			summarizeBy: sum
			sourceColumn: penalties_saved

			annotation SummarizationSetBy = Automatic

		column penalties_missed
			dataType: int64
			formatString: 0
			lineageTag: d921ece8-a38c-48ad-9e41-80a3d92bfaf2
			summarizeBy: sum
			sourceColumn: penalties_missed

			annotation SummarizationSetBy = Automatic

		column yellow_cards
			dataType: int64
			formatString: 0
			lineageTag: f40af689-c097-4f8f-9377-e8bdbe1e68d9
			summarizeBy: sum
			sourceColumn: yellow_cards

			annotation SummarizationSetBy = Automatic

		column red_cards
			dataType: int64
			formatString: 0
			lineageTag: 01fec3a6-12f0-4400-a1d8-3ba13bdf95ea
			summarizeBy: sum
			sourceColumn: red_cards

			annotation SummarizationSetBy = Automatic

		column saves
			dataType: int64
			formatString: 0
			lineageTag: 86fec5f2-671d-451f-9ec1-e80c6c5a373b
			summarizeBy: sum
			sourceColumn: saves

			annotation SummarizationSetBy = Automatic

		column bonus
			dataType: int64
			formatString: 0
			lineageTag: 0d6febc0-1a79-4357-a04b-390ae7b37818
			summarizeBy: sum
			sourceColumn: bonus

			annotation SummarizationSetBy = Automatic

		column bps
			dataType: int64
			formatString: 0
			lineageTag: d923c96f-b96f-4c48-94ef-cc1587318d35
			summarizeBy: sum
			sourceColumn: bps

			annotation SummarizationSetBy = Automatic

		column influence
			dataType: double
			lineageTag: ed72d8c6-8131-42d4-9f4c-03b25e2e5f8e
			summarizeBy: sum
			sourceColumn: influence

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column creativity
			dataType: double
			lineageTag: 4de931ad-c57b-409a-ad74-6ac510bd24df
			summarizeBy: sum
			sourceColumn: creativity

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column threat
			dataType: int64
			formatString: 0
			lineageTag: e15bc7f1-3c08-45cb-a1a8-924c730f0dfe
			summarizeBy: sum
			sourceColumn: threat

			annotation SummarizationSetBy = Automatic

		column ict_index
			dataType: double
			lineageTag: 7cb89868-c4a6-49eb-a16d-ce6760a2560d
			summarizeBy: sum
			sourceColumn: ict_index

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column starts
			dataType: int64
			formatString: 0
			lineageTag: 83b379a0-0fbc-473a-aac9-91c82e2c40ed
			summarizeBy: sum
			sourceColumn: starts

			annotation SummarizationSetBy = Automatic

		column expected_goals
			dataType: double
			lineageTag: 5748a8d6-04ea-4938-b4e7-7f10a8ef2a47
			summarizeBy: sum
			sourceColumn: expected_goals

			annotation SummarizationSetBy = User

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column expected_assists
			dataType: double
			lineageTag: 02fe75df-ed7b-469c-ac8f-297eafe5a0b8
			summarizeBy: sum
			sourceColumn: expected_assists

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column expected_goal_involvements
			dataType: double
			lineageTag: f6d5a7c7-2502-47f2-8ed8-a64e401d2692
			summarizeBy: sum
			sourceColumn: expected_goal_involvements

			annotation SummarizationSetBy = User

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column expected_goals_conceded
			dataType: double
			lineageTag: 2bf3e977-8a58-4e5e-ac77-d2968592d8bb
			summarizeBy: sum
			sourceColumn: expected_goals_conceded

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column value
			dataType: int64
			formatString: 0
			lineageTag: 441cf87f-2ad6-412e-8361-da6e4b11cb2a
			summarizeBy: sum
			sourceColumn: value

			annotation SummarizationSetBy = Automatic

		column transfers_balance
			dataType: int64
			formatString: 0
			lineageTag: e77fbf2a-fc21-4714-ad4a-03f164f13670
			summarizeBy: sum
			sourceColumn: transfers_balance

			annotation SummarizationSetBy = Automatic

		column selected
			dataType: int64
			formatString: 0
			lineageTag: c992b598-cad6-4f91-bfc8-fe16e0e0b8b8
			summarizeBy: sum
			sourceColumn: selected

			annotation SummarizationSetBy = Automatic

		column transfers_in
			dataType: int64
			formatString: 0
			lineageTag: 95e6f75b-4a29-46d9-b786-aee8c2617a71
			summarizeBy: sum
			sourceColumn: transfers_in

			annotation SummarizationSetBy = Automatic

		column transfers_out
			dataType: int64
			formatString: 0
			lineageTag: 5b165883-682d-46b0-9dbd-de56060f501e
			summarizeBy: sum
			sourceColumn: transfers_out

			annotation SummarizationSetBy = Automatic

		column 'Player ID'
			dataType: string
			lineageTag: fc5c287a-56cf-4949-b2f2-9a30aadb34c5
			summarizeBy: none
			sourceColumn: Player ID

			annotation SummarizationSetBy = Automatic

		column 'Position ID'
			dataType: string
			lineageTag: b7fc76f2-70cb-4a59-bef8-ab6e61f0b1ea
			summarizeBy: none
			sourceColumn: Position ID

			annotation SummarizationSetBy = Automatic

		column team_code
			dataType: string
			lineageTag: 814db48a-991a-49a0-a4e4-a4958b870a4f
			summarizeBy: none
			sourceColumn: team_code

			annotation SummarizationSetBy = Automatic

		column 'Kickoff Date'
			dataType: dateTime
			formatString: Long Date
			lineageTag: 554e6f6c-7ebc-4cf2-b9f5-d3819a364f71
			summarizeBy: none
			sourceColumn: Kickoff Date

			annotation SummarizationSetBy = Automatic

			annotation UnderlyingDateTimeDataType = Date

		column 'Club Data (DIM).Team ID'
			dataType: string
			lineageTag: b12d1c68-d402-40f1-9905-209d73e79af0
			summarizeBy: none
			sourceColumn: Club Data (DIM).Team ID

			annotation SummarizationSetBy = Automatic

		column 'Club Data (DIM).Team name'
			dataType: string
			lineageTag: 4b0f4022-943b-469a-bfd5-503ad141bbf6
			summarizeBy: none
			sourceColumn: Club Data (DIM).Team name

			annotation SummarizationSetBy = Automatic

		column modified
			dataType: string
			lineageTag: 613094c5-0c46-42ba-b863-690c8c2df5a9
			summarizeBy: none
			sourceColumn: modified

			annotation SummarizationSetBy = Automatic

		column Price
			dataType: decimal
			formatString: "£"#,0.###############;-"£"#,0.###############;"£"#,0.###############
			lineageTag: 7748a6e9-34fa-418c-ab0e-cd361f69f89b
			summarizeBy: sum
			sourceColumn: Price

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"currencyCulture":"en-GB"}

		column xGCDiff = 'Detailed Player Data (Fact)'[Goals Scored] - 'Detailed Player Data (Fact)'[expected_goals]
			lineageTag: e60d7216-1f51-4927-8820-e8e58cf6a0a7
			summarizeBy: sum

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column clearances_blocks_interceptions
			dataType: int64
			formatString: 0
			lineageTag: 6c674473-2925-467a-af23-8d29b2bcf1bc
			summarizeBy: sum
			sourceColumn: clearances_blocks_interceptions

			annotation SummarizationSetBy = Automatic

		column recoveries
			dataType: string
			lineageTag: fb2cad19-07bc-442f-91a8-1ba593e504da
			summarizeBy: none
			sourceColumn: recoveries

			annotation SummarizationSetBy = Automatic

		column tackles
			dataType: string
			lineageTag: fe342883-6175-4728-9801-404f27288d6f
			summarizeBy: none
			sourceColumn: tackles

			annotation SummarizationSetBy = Automatic

		column defensive_contribution
			dataType: int64
			formatString: 0
			lineageTag: ee55c650-4ffb-418f-8196-955f7b23a658
			summarizeBy: sum
			sourceColumn: defensive_contribution

			annotation SummarizationSetBy = Automatic

		column Column1
			dataType: string
			lineageTag: 5de059a7-f388-4fc4-92dd-9bbc0850feee
			summarizeBy: none
			sourceColumn: Column1

			annotation SummarizationSetBy = Automatic

		partition 'Detailed Player Data (Fact)-a038f8e5-2e14-478e-b159-9e6ddfcdec92' = m
			mode: import
			source =
					let
					    Source = Csv.Document(File.Contents("C:\Users\GethynEllis\OneDrive - GRE Solutions Limited\PHIT Football\Data\PLayerDataLoopTest.csv"),[Delimiter=",", Columns=45, Encoding=1252, QuoteStyle=QuoteStyle.None]),
					    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}, {"Column8", type text}, {"Column9", type text}, {"Column10", type text}, {"Column11", type text}, {"Column12", type text}, {"Column13", type text}, {"Column14", type text}, {"Column15", type text}, {"Column16", type text}, {"Column17", type text}, {"Column18", type text}, {"Column19", type text}, {"Column20", type text}, {"Column21", type text}, {"Column22", type text}, {"Column23", type text}, {"Column24", type text}, {"Column25", type text}, {"Column26", type text}, {"Column27", type text}, {"Column28", type text}, {"Column29", type text}, {"Column30", type text}, {"Column31", type text}, {"Column32", type text}, {"Column33", type text}, {"Column34", type text}, {"Column35", type text}, {"Column36", type text}, {"Column37", type text}}),
					    #"Removed Top Rows" = Table.Skip(#"Changed Type",1),
					    #"Promoted Headers" = Table.PromoteHeaders(#"Removed Top Rows", [PromoteAllScalars=true]),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"element", Int64.Type}, {"fixture", Int64.Type}, {"opponent_team", Int64.Type}, {"total_points", Int64.Type}, {"was_home", type logical}, {"kickoff_time", type datetime}, {"team_h_score", Int64.Type}, {"team_a_score", Int64.Type}, {"round", Int64.Type}, {"minutes", Int64.Type}, {"goals_scored", Int64.Type}, {"assists", Int64.Type}, {"clean_sheets", Int64.Type}, {"goals_conceded", Int64.Type}, {"own_goals", Int64.Type}, {"penalties_saved", Int64.Type}, {"penalties_missed", Int64.Type}, {"yellow_cards", Int64.Type}, {"red_cards", Int64.Type}, {"saves", Int64.Type}, {"bonus", Int64.Type}, {"bps", Int64.Type}, {"influence", type number}, {"creativity", type number}, {"threat", Int64.Type}, {"ict_index", type number}, {"starts", Int64.Type}, {"expected_goals", type number}, {"expected_assists", type number}, {"expected_goal_involvements", type number}, {"expected_goals_conceded", type number}, {"value", Int64.Type}, {"transfers_balance", Int64.Type}, {"selected", Int64.Type}, {"transfers_in", Int64.Type}, {"transfers_out", Int64.Type}, {"ID", type text}}),
					    #"Renamed Columns" = Table.RenameColumns(#"Changed Type1",{{"ID", "Player ID"}}),
					    #"Merged Queries" = Table.NestedJoin(#"Renamed Columns", {"element"}, #"Master Data", {"id"}, "Master Data", JoinKind.LeftOuter),
					    #"Expanded Master Data" = Table.ExpandTableColumn(#"Merged Queries", "Master Data", {"Position ID", "team_code"}, {"Position ID", "team_code"}),
					    #"Renamed Columns1" = Table.RenameColumns(#"Expanded Master Data",{{"fixture", "Fixture ID"}}),
					    #"Changed Type2" = Table.TransformColumnTypes(#"Renamed Columns1",{{"Fixture ID", type text}, {"round", type text}}),
					    #"Renamed Columns2" = Table.RenameColumns(#"Changed Type2",{{"round", "Gameweek ID"}}),
					    #"Duplicated Column" = Table.DuplicateColumn(#"Renamed Columns2", "kickoff_time", "kickoff_time - Copy"),
					    #"Renamed Columns3" = Table.RenameColumns(#"Duplicated Column",{{"kickoff_time - Copy", "Kickoff Date"}}),
					    #"Changed Type3" = Table.TransformColumnTypes(#"Renamed Columns3",{{"Kickoff Date", type date}}),
					    #"Renamed Columns4" = Table.RenameColumns(#"Changed Type3",{{"element", "Player"}, {"total_points", "Total Points"}, {"opponent_team", "Opposition Team ID"}, {"team_h_score", "Home Teams Score"}, {"team_a_score", "Away Team Score"}, {"minutes", "Minutes played"}, {"goals_scored", "Goals Scored"}, {"assists", "Assists made"}, {"clean_sheets", "Clean sheets"}, {"goals_conceded", "Goals conceded"}, {"own_goals", "Own goals"}}),
					    #"Appended Query" = Table.Combine({#"Renamed Columns4", OwnGoals}),
					    #"Duplicated Column1" = Table.DuplicateColumn(#"Appended Query", "value", "value - Copy"),
					    #"Renamed Columns5" = Table.RenameColumns(#"Duplicated Column1",{{"value - Copy", "Price - $Mill"}}),
					    #"Removed Columns" = Table.RemoveColumns(#"Renamed Columns5",{"Price - $Mill"}),
					    #"Added Custom" = Table.AddColumn(#"Removed Columns", "Price", each ([value]/10) * 1000000),
					    #"Changed Type4" = Table.TransformColumnTypes(#"Added Custom",{{"Price", Currency.Type}}),
					    #"Sorted Rows" = Table.Sort(#"Changed Type4",{{"Price", Order.Descending}}),
					    #"Filtered Rows" = Table.SelectRows(#"Sorted Rows", each true),
					    #"Removed Columns1" = Table.RemoveColumns(#"Filtered Rows",{"_1", "_2"}),
					    #"Changed Type5" = Table.TransformColumnTypes(#"Removed Columns1",{{"defensive_contribution", Int64.Type}, {"clearances_blocks_interceptions", Int64.Type}})
					in
					    #"Changed Type5"

		annotation PBI_ResultType = Table

		annotation PBI_NavigationStepName = Navigation




---

createOrReplace

	table 'Fixtures (DIM)'
		lineageTag: f166cb28-957a-42e8-bda0-1262ec3180cf

		column code
			dataType: string
			lineageTag: 00b373bd-78bb-4ea3-8ad0-4b4fdf98f2ec
			summarizeBy: none
			sourceColumn: code

			annotation SummarizationSetBy = Automatic

		column event
			dataType: string
			lineageTag: d5706b91-559b-4dee-8535-ec4d3fa8485c
			summarizeBy: none
			sourceColumn: event

			annotation SummarizationSetBy = Automatic

		column finished
			dataType: string
			lineageTag: 66d29cd9-dc9e-45b6-808d-8f2fe146639c
			summarizeBy: none
			sourceColumn: finished

			annotation SummarizationSetBy = Automatic

		column finished_provisional
			dataType: string
			lineageTag: 6bc4cf1b-85ec-426b-a76d-0b43dc83c972
			summarizeBy: none
			sourceColumn: finished_provisional

			annotation SummarizationSetBy = Automatic

		column id
			dataType: string
			lineageTag: 479680a2-dac5-4fb3-9778-274b645fe3b4
			summarizeBy: none
			sourceColumn: id

			annotation SummarizationSetBy = Automatic

		column kickoff_time
			dataType: string
			lineageTag: 0e1f1a4e-184b-4aa8-95ef-f9b5c597a15c
			summarizeBy: none
			sourceColumn: kickoff_time

			annotation SummarizationSetBy = Automatic

		column minutes
			dataType: string
			lineageTag: 5b6c16d0-ccc9-445c-9e1a-e86a7b1a6623
			summarizeBy: none
			sourceColumn: minutes

			annotation SummarizationSetBy = Automatic

		column provisional_start_time
			dataType: string
			lineageTag: 7f45b5c7-6a57-4835-8f17-83b033e755e2
			summarizeBy: none
			sourceColumn: provisional_start_time

			annotation SummarizationSetBy = Automatic

		column started
			dataType: string
			lineageTag: 45b6b03c-9cfc-4810-bb6c-5dbd80d84a81
			summarizeBy: none
			sourceColumn: started

			annotation SummarizationSetBy = Automatic

		column team_a
			dataType: string
			lineageTag: c4a33ee4-6a37-4d67-ad10-fb7802ff5ab1
			summarizeBy: none
			sourceColumn: team_a

			annotation SummarizationSetBy = Automatic

		column team_a_score
			dataType: string
			lineageTag: ac7c9dfe-221f-4a7d-b366-3232bc237ffc
			summarizeBy: none
			sourceColumn: team_a_score

			annotation SummarizationSetBy = Automatic

		column team_h
			dataType: string
			lineageTag: 885b43b3-c68f-45af-aa7f-5cbdeb1ea1c8
			summarizeBy: none
			sourceColumn: team_h

			annotation SummarizationSetBy = Automatic

		column team_h_score
			dataType: string
			lineageTag: 313adff3-8897-4811-aa26-b1e40974a8e3
			summarizeBy: none
			sourceColumn: team_h_score

			annotation SummarizationSetBy = Automatic

		column stats
			dataType: string
			lineageTag: 37ff2df3-405a-42a1-aea0-b1b27a16316d
			summarizeBy: none
			sourceColumn: stats

			annotation SummarizationSetBy = Automatic

		column team_h_difficulty
			dataType: string
			lineageTag: eec22113-1486-4a1d-bc97-4f14ab9af9d5
			summarizeBy: none
			sourceColumn: team_h_difficulty

			annotation SummarizationSetBy = Automatic

		column team_a_difficulty
			dataType: string
			lineageTag: 6fef37d6-61b7-4e51-9820-abb9149bd7dd
			summarizeBy: none
			sourceColumn: team_a_difficulty

			annotation SummarizationSetBy = Automatic

		column pulse_id
			dataType: string
			lineageTag: f7e84fb9-d008-41d0-8ed1-a6d7cba56157
			summarizeBy: none
			sourceColumn: pulse_id

			annotation SummarizationSetBy = Automatic

		partition 'Fixtures (DIM)' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents(" https://fantasy.premierleague.com/api/fixtures/")),
					    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"code", "event", "finished", "finished_provisional", "id", "kickoff_time", "minutes", "provisional_start_time", "started", "team_a", "team_a_score", "team_h", "team_h_score", "stats", "team_h_difficulty", "team_a_difficulty", "pulse_id"}, {"code", "event", "finished", "finished_provisional", "id", "kickoff_time", "minutes", "provisional_start_time", "started", "team_a", "team_a_score", "team_h", "team_h_score", "stats", "team_h_difficulty", "team_a_difficulty", "pulse_id"}),
					    #"Filtered Rows" = Table.SelectRows(#"Expanded Column1", each ([finished] = false))
					in
					    #"Filtered Rows"

		annotation PBI_ResultType = Table



---

createOrReplace

	table GWDim
		lineageTag: a8f33ca6-a22e-4424-99fb-ba200a3fe7f9

		column 'GW ID'
			dataType: int64
			formatString: 0
			lineageTag: 4cbfd365-d261-44b0-8909-d12d48ecc44f
			summarizeBy: none
			sourceColumn: GW ID

			annotation SummarizationSetBy = Automatic

		column 'GW Name'
			dataType: string
			lineageTag: 1f5952b4-887a-4548-b816-6b2232d26b6b
			summarizeBy: none
			sourceColumn: GW Name

			annotation SummarizationSetBy = Automatic

		column 'GW Deadline'
			dataType: dateTime
			formatString: General Date
			lineageTag: bac01ecc-3df0-4c1d-b9f4-dc4dad00f1af
			summarizeBy: none
			sourceColumn: GW Deadline

			variation Variation
				isDefault
				relationship: f960787a-7df0-4797-a966-11972e765210
				defaultHierarchy: LocalDateTable_ad217237-dc5f-4a13-afdc-804ba9d472e1.'Date Hierarchy'

			annotation SummarizationSetBy = Automatic

		column Finished
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: ac7c4f5a-c536-44c8-ae69-cd73ef4dc861
			summarizeBy: none
			sourceColumn: Finished

			annotation SummarizationSetBy = Automatic

		column is_previous
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: e6c3a8cd-43cf-4f37-9505-7898746d3d19
			summarizeBy: none
			sourceColumn: is_previous

			annotation SummarizationSetBy = Automatic

		column is_current
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: 13d71eba-1141-4cb5-a49d-c1fd52ffa0b6
			summarizeBy: none
			sourceColumn: is_current

			annotation SummarizationSetBy = Automatic

		column is_next
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: 227d2697-891f-4d17-abc8-2fc27c7b2cb0
			summarizeBy: none
			sourceColumn: is_next

			annotation SummarizationSetBy = Automatic

		partition GWDim = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    #"Converted to Table" = Table.FromRecords({Source}),
					    #"Expanded events" = Table.ExpandListColumn(#"Converted to Table", "events"),
					    #"Expanded events1" = Table.ExpandRecordColumn(#"Expanded events", "events", {"id", "name", "deadline_time", "release_time", "average_entry_score", "finished", "data_checked", "highest_scoring_entry", "deadline_time_epoch", "deadline_time_game_offset", "highest_score", "is_previous", "is_current", "is_next", "cup_leagues_created", "h2h_ko_matches_created", "can_enter", "can_manage", "released", "ranked_count", "overrides", "chip_plays", "most_selected", "most_transferred_in", "top_element", "top_element_info", "transfers_made", "most_captained", "most_vice_captained"}, {"events.id", "events.name", "events.deadline_time", "events.release_time", "events.average_entry_score", "events.finished", "events.data_checked", "events.highest_scoring_entry", "events.deadline_time_epoch", "events.deadline_time_game_offset", "events.highest_score", "events.is_previous", "events.is_current", "events.is_next", "events.cup_leagues_created", "events.h2h_ko_matches_created", "events.can_enter", "events.can_manage", "events.released", "events.ranked_count", "events.overrides", "events.chip_plays", "events.most_selected", "events.most_transferred_in", "events.top_element", "events.top_element_info", "events.transfers_made", "events.most_captained", "events.most_vice_captained"}),
					    #"Removed Other Columns" = Table.SelectColumns(#"Expanded events1",{"events.id", "events.name", "events.deadline_time", "events.finished", "events.is_previous", "events.is_current", "events.is_next"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Removed Other Columns",{{"events.deadline_time", type datetime}}),
					    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"events.id", "GW ID"}, {"events.name", "GW Name"}, {"events.deadline_time", "GW Deadline"}, {"events.finished", "Finished"}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"Finished", type logical}}),
					    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"events.is_previous", "is_previous"}}),
					    #"Changed Type2" = Table.TransformColumnTypes(#"Renamed Columns1",{{"is_previous", type logical}, {"events.is_current", type logical}}),
					    #"Renamed Columns2" = Table.RenameColumns(#"Changed Type2",{{"events.is_current", "is_current"}, {"events.is_next", "is_next"}}),
					    #"Changed Type3" = Table.TransformColumnTypes(#"Renamed Columns2",{{"is_next", type logical}, {"GW ID", type date}}),
					    #"Sorted Rows" = Table.Sort(#"Changed Type3",{{"GW Deadline", Order.Ascending}}),
					    #"Changed Type4" = Table.TransformColumnTypes(#"Sorted Rows",{{"GW ID", Int64.Type}})
					in
					    #"Changed Type4"

		annotation PBI_NavigationStepName = Navigation

		annotation PBI_ResultType = Table




---

createOrReplace

	table 'Opposition Club Data (DIM)'
		lineageTag: 281fef90-1a07-421d-b00c-a2dff57f2f5b

		column 'Team Code'
			dataType: string
			lineageTag: 2d4eedd8-46be-4822-b425-1f975324683e
			summarizeBy: none
			sourceColumn: Team Code

			annotation SummarizationSetBy = Automatic

		column 'Team ID'
			dataType: string
			lineageTag: 810985c2-b4a9-4534-a06d-6de4bb758f22
			summarizeBy: none
			sourceColumn: Team ID

			annotation SummarizationSetBy = Automatic

		column 'Team name'
			dataType: string
			lineageTag: c2faaa7f-967e-43a0-b4d0-77dc6f12decf
			summarizeBy: none
			sourceColumn: Team name

			annotation SummarizationSetBy = Automatic

		column pulse_id
			dataType: string
			lineageTag: 45d663da-b4ad-4751-87d0-707ae8719297
			summarizeBy: none
			sourceColumn: pulse_id

			annotation SummarizationSetBy = Automatic

		partition 'Opposition Club Data (DIM)' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    teams = Source[teams],
					    #"Converted to Table" = Table.FromList(teams, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"code", "draw", "form", "id", "loss", "name", "played", "points", "position", "short_name", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away", "pulse_id"}, {"code", "draw", "form", "id", "loss", "name", "played", "points", "position", "short_name", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away", "pulse_id"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Column1",{{"code", type text}}),
					    #"Removed Columns" = Table.RemoveColumns(#"Changed Type",{"draw", "form", "loss", "played", "points", "position", "strength", "team_division", "unavailable", "win", "strength_overall_home", "strength_overall_away", "strength_attack_home", "strength_attack_away", "strength_defence_home", "strength_defence_away"}),
					    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{{"id", "Team ID"}, {"code", "Team Code"}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"Team ID", type text}}),
					    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"name", "Team name"}}),
					    #"Removed Columns1" = Table.RemoveColumns(#"Renamed Columns1",{"short_name"})
					in
					    #"Removed Columns1"

		annotation PBI_ResultType = Table

---

createOrReplace

	table 'Player Data (Dim)'
		lineageTag: 2e9e04bf-6d59-449c-b205-d29bdadb8238

		column 'Player ID'
			dataType: string
			lineageTag: 6d0edf4b-9d7c-4f90-8c93-470464efd73d
			summarizeBy: none
			sourceColumn: Player ID

			annotation SummarizationSetBy = Automatic

		column first_name
			dataType: string
			lineageTag: 4a9a18a6-cbdb-4d73-9dc7-031f662461e6
			summarizeBy: none
			sourceColumn: first_name

			annotation SummarizationSetBy = Automatic

		column second_name
			dataType: string
			lineageTag: 2f24e9e9-576e-4a0e-8133-74d164c9562a
			summarizeBy: none
			sourceColumn: second_name

			annotation SummarizationSetBy = Automatic

		column now_cost
			dataType: decimal
			formatString: "£"#,0.###############;-"£"#,0.###############;"£"#,0.###############
			lineageTag: 1d8a10f0-da67-47ab-8ef7-80fec79e5d66
			summarizeBy: none
			sourceColumn: now_cost

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"currencyCulture":"en-GB"}

		column 'Player Name'
			dataType: string
			lineageTag: 72c9ae95-4bad-445f-b956-e2e5a5781491
			summarizeBy: none
			sourceColumn: Player Name

			annotation SummarizationSetBy = Automatic

		column ep_next
			dataType: double
			lineageTag: b5f94af8-2c6d-4ea1-9fcb-6efcb947a50e
			summarizeBy: none
			sourceColumn: ep_next

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column DemoPLayerName = CONCATENATE( 'Player Data (Dim)'[first_name], 'Player Data (Dim)'[second_name])
			lineageTag: 51fe6e1c-3f9c-413d-8fcb-36a1d4d35fa7
			summarizeBy: none

			annotation SummarizationSetBy = Automatic

		column NowCostMillions
			dataType: decimal
			formatString: "£"#,0.###############;-"£"#,0.###############;"£"#,0.###############
			lineageTag: 1ce69bd4-28fd-4346-9036-a71391e736a0
			summarizeBy: sum
			sourceColumn: NowCostMillions

			annotation SummarizationSetBy = User

			annotation PBI_FormatHint = {"currencyCulture":"en-GB"}

		partition 'Player Data (Dim)-0ce30944-7218-4236-bba5-85c4e4a69127' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    elements = Source[elements],
					    #"Converted to Table" = Table.FromList(elements, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"chance_of_playing_next_round", "chance_of_playing_this_round", "code", "cost_change_event", "cost_change_event_fall", "cost_change_start", "cost_change_start_fall", "dreamteam_count", "element_type", "ep_next", "ep_this", "event_points", "first_name", "form", "id", "in_dreamteam", "news", "news_added", "now_cost", "photo", "points_per_game", "second_name", "selected_by_percent", "special", "squad_number", "status", "team", "team_code", "total_points", "transfers_in", "transfers_in_event", "transfers_out", "transfers_out_event", "value_form", "value_season", "web_name", "minutes", "goals_scored", "assists", "clean_sheets", "goals_conceded", "own_goals", "penalties_saved", "penalties_missed", "yellow_cards", "red_cards", "saves", "bonus", "bps", "influence", "creativity", "threat", "ict_index", "starts", "expected_goals", "expected_assists", "expected_goal_involvements", "expected_goals_conceded", "influence_rank", "influence_rank_type", "creativity_rank", "creativity_rank_type", "threat_rank", "threat_rank_type", "ict_index_rank", "ict_index_rank_type", "corners_and_indirect_freekicks_order", "corners_and_indirect_freekicks_text", "direct_freekicks_order", "direct_freekicks_text", "penalties_order", "penalties_text", "expected_goals_per_90", "saves_per_90", "expected_assists_per_90", "expected_goal_involvements_per_90", "expected_goals_conceded_per_90", "goals_conceded_per_90", "now_cost_rank", "now_cost_rank_type", "form_rank", "form_rank_type", "points_per_game_rank", "points_per_game_rank_type", "selected_rank", "selected_rank_type", "starts_per_90", "clean_sheets_per_90"}, {"chance_of_playing_next_round", "chance_of_playing_this_round", "code", "cost_change_event", "cost_change_event_fall", "cost_change_start", "cost_change_start_fall", "dreamteam_count", "element_type", "ep_next", "ep_this", "event_points", "first_name", "form", "id", "in_dreamteam", "news", "news_added", "now_cost", "photo", "points_per_game", "second_name", "selected_by_percent", "special", "squad_number", "status", "team", "team_code", "total_points", "transfers_in", "transfers_in_event", "transfers_out", "transfers_out_event", "value_form", "value_season", "web_name", "minutes", "goals_scored", "assists", "clean_sheets", "goals_conceded", "own_goals", "penalties_saved", "penalties_missed", "yellow_cards", "red_cards", "saves", "bonus", "bps", "influence", "creativity", "threat", "ict_index", "starts", "expected_goals", "expected_assists", "expected_goal_involvements", "expected_goals_conceded", "influence_rank", "influence_rank_type", "creativity_rank", "creativity_rank_type", "threat_rank", "threat_rank_type", "ict_index_rank", "ict_index_rank_type", "corners_and_indirect_freekicks_order", "corners_and_indirect_freekicks_text", "direct_freekicks_order", "direct_freekicks_text", "penalties_order", "penalties_text", "expected_goals_per_90", "saves_per_90", "expected_assists_per_90", "expected_goal_involvements_per_90", "expected_goals_conceded_per_90", "goals_conceded_per_90", "now_cost_rank", "now_cost_rank_type", "form_rank", "form_rank_type", "points_per_game_rank", "points_per_game_rank_type", "selected_rank", "selected_rank_type", "starts_per_90", "clean_sheets_per_90"}),
					    #"Removed Other Columns" = Table.SelectColumns(#"Expanded Column1",{"ep_next", "first_name", "id", "now_cost", "second_name", "web_name"}),
					    #"Changed Type3" = Table.TransformColumnTypes(#"Removed Other Columns",{{"ep_next", type number}}),
					    #"Reordered Columns" = Table.ReorderColumns(#"Changed Type3",{"first_name", "second_name", "id", "now_cost", "web_name"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Reordered Columns",{{"id", type text}}),
					    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"id", "Player ID"}}),
					    #"Removed Columns" = Table.RemoveColumns(#"Renamed Columns",{"web_name"}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Removed Columns",{{"now_cost", type text}, {"first_name", type text}}),
					    #"Added Custom" = Table.AddColumn(#"Changed Type1", "Custom", each Text.Combine ({[first_name], [second_name]}, " ")),
					    #"Renamed Columns1" = Table.RenameColumns(#"Added Custom",{{"Custom", "Player Name"}}),
					    #"Changed Type2" = Table.TransformColumnTypes(#"Renamed Columns1",{{"now_cost", Currency.Type}, {"Player ID", type text}, {"second_name", type text}, {"Player Name", type text}}),
					    #"Added Custom1" = Table.AddColumn(#"Changed Type2", "NowCostMillions", each [now_cost]/10),
					    #"Changed Type4" = Table.TransformColumnTypes(#"Added Custom1",{{"NowCostMillions", Currency.Type}})
					in
					    #"Changed Type4"

		annotation PBI_ResultType = Table

		annotation PBI_NavigationStepName = Navigation




---

createOrReplace

	table 'Player Summary'
		lineageTag: 2fb268df-4ee9-40da-86dc-136c56bfc139

		column chance_of_playing_next_round
			dataType: string
			lineageTag: e384edfb-ecea-4939-beaa-ebc7655419c2
			summarizeBy: none
			sourceColumn: chance_of_playing_next_round

			annotation SummarizationSetBy = Automatic

		column chance_of_playing_this_round
			dataType: string
			lineageTag: 39c1692a-dec5-4ccc-afd4-944404485f74
			summarizeBy: none
			sourceColumn: chance_of_playing_this_round

			annotation SummarizationSetBy = Automatic

		column code
			dataType: string
			lineageTag: 50b44da9-754b-412a-90f2-ec97418c92fc
			summarizeBy: none
			sourceColumn: code

			annotation SummarizationSetBy = Automatic

		column cost_change_event
			dataType: string
			lineageTag: 45b8dedc-ba60-4078-a554-d9b01f16268b
			summarizeBy: none
			sourceColumn: cost_change_event

			annotation SummarizationSetBy = Automatic

		column cost_change_event_fall
			dataType: string
			lineageTag: a960addc-38d9-4a0b-8252-9fe1974fe8ae
			summarizeBy: none
			sourceColumn: cost_change_event_fall

			annotation SummarizationSetBy = Automatic

		column cost_change_start
			dataType: string
			lineageTag: 64b855a0-5d5b-4743-a18c-0f05b5d1edf5
			summarizeBy: none
			sourceColumn: cost_change_start

			annotation SummarizationSetBy = Automatic

		column cost_change_start_fall
			dataType: string
			lineageTag: 3df13543-6faa-4751-9a1d-95e444e2f505
			summarizeBy: none
			sourceColumn: cost_change_start_fall

			annotation SummarizationSetBy = Automatic

		column dreamteam_count
			dataType: string
			lineageTag: 94137a2d-6f47-4771-b726-334f62e10524
			summarizeBy: none
			sourceColumn: dreamteam_count

			annotation SummarizationSetBy = Automatic

		column 'Position ID'
			dataType: string
			lineageTag: 4c29d8f7-dee9-47ce-856b-e877290abe28
			summarizeBy: none
			sourceColumn: Position ID

			annotation SummarizationSetBy = Automatic

		column ep_next
			dataType: string
			lineageTag: f27d43c4-5605-47f5-83ef-51e5592211de
			summarizeBy: none
			sourceColumn: ep_next

			annotation SummarizationSetBy = Automatic

		column ep_this
			dataType: string
			lineageTag: d91df1e9-98ef-4e68-a355-d04aad97603c
			summarizeBy: none
			sourceColumn: ep_this

			annotation SummarizationSetBy = Automatic

		column event_points
			dataType: string
			lineageTag: b6f97570-db08-446e-aa23-37c3c165fad1
			summarizeBy: none
			sourceColumn: event_points

			annotation SummarizationSetBy = Automatic

		column first_name
			dataType: string
			lineageTag: 656b7833-ba3e-4826-aa6b-f69351b67a60
			summarizeBy: none
			sourceColumn: first_name

			annotation SummarizationSetBy = Automatic

		column form
			dataType: string
			lineageTag: 8532905b-c3fb-41f2-9862-2ce1d65acf4f
			summarizeBy: none
			sourceColumn: form

			annotation SummarizationSetBy = Automatic

		column id
			dataType: string
			lineageTag: a13f0cf7-e41d-47f5-959a-4bd106bcb775
			summarizeBy: none
			sourceColumn: id

			annotation SummarizationSetBy = Automatic

		column in_dreamteam
			dataType: string
			lineageTag: f3a3a370-3acd-434d-b431-5f822d02b98f
			summarizeBy: none
			sourceColumn: in_dreamteam

			annotation SummarizationSetBy = Automatic

		column news
			dataType: string
			lineageTag: e13dbf17-eef3-49e8-9d9c-de4a04481230
			summarizeBy: none
			sourceColumn: news

			annotation SummarizationSetBy = Automatic

		column news_added
			dataType: string
			lineageTag: eba256d2-fd31-43d5-a9db-d3c25ab40578
			summarizeBy: none
			sourceColumn: news_added

			annotation SummarizationSetBy = Automatic

		column now_cost
			dataType: string
			lineageTag: 6be2d413-aa34-434e-b8c7-cf1e0b1eeda0
			summarizeBy: none
			sourceColumn: now_cost

			annotation SummarizationSetBy = Automatic

		column photo
			dataType: string
			lineageTag: 2cc60858-48b4-4ce0-9bf0-c835e86639de
			summarizeBy: none
			sourceColumn: photo

			annotation SummarizationSetBy = Automatic

		column points_per_game
			dataType: string
			lineageTag: 7c549cca-1f40-4f86-b5fa-4a02485b4023
			summarizeBy: none
			sourceColumn: points_per_game

			annotation SummarizationSetBy = Automatic

		column second_name
			dataType: string
			lineageTag: b707e20c-0c1e-46a6-9fae-1832297ab648
			summarizeBy: none
			sourceColumn: second_name

			annotation SummarizationSetBy = Automatic

		column selected_by_percent
			dataType: double
			lineageTag: ec73fe27-62b5-4fe9-bffc-98e89aa5ecdd
			summarizeBy: sum
			sourceColumn: selected_by_percent

			annotation SummarizationSetBy = Automatic

			annotation PBI_FormatHint = {"isGeneralNumber":true}

		column special
			dataType: string
			lineageTag: ffa4de68-63de-4184-b4f8-d1502394e9da
			summarizeBy: none
			sourceColumn: special

			annotation SummarizationSetBy = Automatic

		column squad_number
			dataType: string
			lineageTag: c55fd9a0-d7d4-466f-b145-de9fae07a1a5
			summarizeBy: none
			sourceColumn: squad_number

			annotation SummarizationSetBy = Automatic

		column status
			dataType: string
			lineageTag: 02c0b752-8690-4d2d-b8cd-f97beda9629e
			summarizeBy: none
			sourceColumn: status

			annotation SummarizationSetBy = Automatic

		column team
			dataType: string
			lineageTag: a05821d8-45a9-4a89-b7ee-949ee09e5260
			summarizeBy: none
			sourceColumn: team

			annotation SummarizationSetBy = Automatic

		column team_code
			dataType: string
			lineageTag: 567708fa-c045-44b0-9e0f-d83a03041fe2
			summarizeBy: none
			sourceColumn: team_code

			annotation SummarizationSetBy = Automatic

		column total_points
			dataType: string
			lineageTag: 4b45a262-e7d9-4ae4-a7a9-6bc4578957d5
			summarizeBy: none
			sourceColumn: total_points

			annotation SummarizationSetBy = Automatic

		column transfers_in
			dataType: string
			lineageTag: aded918f-8d4a-4f46-bbd8-a09ecee27458
			summarizeBy: none
			sourceColumn: transfers_in

			annotation SummarizationSetBy = Automatic

		column transfers_in_event
			dataType: string
			lineageTag: 00e83beb-bc6f-4d48-8370-5aed47950ba4
			summarizeBy: none
			sourceColumn: transfers_in_event

			annotation SummarizationSetBy = Automatic

		column transfers_out
			dataType: string
			lineageTag: 27df5f1a-65c5-45e6-9f7a-ab08e60f4926
			summarizeBy: none
			sourceColumn: transfers_out

			annotation SummarizationSetBy = Automatic

		column transfers_out_event
			dataType: string
			lineageTag: 3b93661f-70af-4cb4-8554-33a4986d9460
			summarizeBy: none
			sourceColumn: transfers_out_event

			annotation SummarizationSetBy = Automatic

		column value_form
			dataType: string
			lineageTag: 36c3209d-5502-4ad2-b90f-383495439a7c
			summarizeBy: none
			sourceColumn: value_form

			annotation SummarizationSetBy = Automatic

		column value_season
			dataType: string
			lineageTag: 66edab85-ad1c-409a-914d-c7d62992023c
			summarizeBy: none
			sourceColumn: value_season

			annotation SummarizationSetBy = Automatic

		column web_name
			dataType: string
			lineageTag: 228d1f5c-f47f-498a-9935-4bd7510ea7db
			summarizeBy: none
			sourceColumn: web_name

			annotation SummarizationSetBy = Automatic

		column minutes
			dataType: string
			lineageTag: b4422d07-c16b-4a5f-b236-a5d82522e3f9
			summarizeBy: none
			sourceColumn: minutes

			annotation SummarizationSetBy = Automatic

		column goals_scored
			dataType: string
			lineageTag: 043cf604-ecc5-44b1-a6ba-71bc3de8cc84
			summarizeBy: none
			sourceColumn: goals_scored

			annotation SummarizationSetBy = Automatic

		column assists
			dataType: string
			lineageTag: 3a0e1aa6-e654-4a48-8ca6-743117a33aef
			summarizeBy: none
			sourceColumn: assists

			annotation SummarizationSetBy = Automatic

		column clean_sheets
			dataType: string
			lineageTag: f448be0c-4383-434a-b41c-8c95a19178c9
			summarizeBy: none
			sourceColumn: clean_sheets

			annotation SummarizationSetBy = Automatic

		column goals_conceded
			dataType: string
			lineageTag: d37e6cd8-985a-48b1-aa3a-fb893c39b494
			summarizeBy: none
			sourceColumn: goals_conceded

			annotation SummarizationSetBy = Automatic

		column own_goals
			dataType: string
			lineageTag: 58f385b2-43b6-443e-a474-e23b810afd4d
			summarizeBy: none
			sourceColumn: own_goals

			annotation SummarizationSetBy = Automatic

		column penalties_saved
			dataType: string
			lineageTag: 9103a05a-3a66-41ac-9518-0cb34683cd2c
			summarizeBy: none
			sourceColumn: penalties_saved

			annotation SummarizationSetBy = Automatic

		column penalties_missed
			dataType: string
			lineageTag: 314d13bc-3c44-4e64-9c24-7f7c1cf56a5c
			summarizeBy: none
			sourceColumn: penalties_missed

			annotation SummarizationSetBy = Automatic

		column yellow_cards
			dataType: string
			lineageTag: 6ac256c1-9b17-4de6-bd34-edda36024bc9
			summarizeBy: none
			sourceColumn: yellow_cards

			annotation SummarizationSetBy = Automatic

		column red_cards
			dataType: string
			lineageTag: 96b7e2a1-030b-4970-9ee1-c7fb48f7d7da
			summarizeBy: none
			sourceColumn: red_cards

			annotation SummarizationSetBy = Automatic

		column saves
			dataType: string
			lineageTag: 6109ebfa-320a-4979-a604-d8ffaba6b7c4
			summarizeBy: none
			sourceColumn: saves

			annotation SummarizationSetBy = Automatic

		column bonus
			dataType: string
			lineageTag: ae82d5d2-65e6-4854-bb46-5dc7c6be1d53
			summarizeBy: none
			sourceColumn: bonus

			annotation SummarizationSetBy = Automatic

		column bps
			dataType: string
			lineageTag: 4ae43e57-4655-469c-bb64-289ddcff0ef1
			summarizeBy: none
			sourceColumn: bps

			annotation SummarizationSetBy = Automatic

		column influence
			dataType: string
			lineageTag: fca1f8cf-3932-4fb9-995a-8e06f298889a
			summarizeBy: none
			sourceColumn: influence

			annotation SummarizationSetBy = Automatic

		column creativity
			dataType: string
			lineageTag: 4216ceb7-d573-4241-8138-1c98a4379918
			summarizeBy: none
			sourceColumn: creativity

			annotation SummarizationSetBy = Automatic

		column threat
			dataType: string
			lineageTag: 98f2af03-7c1a-472f-89a4-02388ad1445c
			summarizeBy: none
			sourceColumn: threat

			annotation SummarizationSetBy = Automatic

		column ict_index
			dataType: string
			lineageTag: 7767622b-4316-44e6-91c6-2ce89467c188
			summarizeBy: none
			sourceColumn: ict_index

			annotation SummarizationSetBy = Automatic

		column starts
			dataType: string
			lineageTag: cfe765d8-e914-4c23-ba81-e96c8451f4fc
			summarizeBy: none
			sourceColumn: starts

			annotation SummarizationSetBy = Automatic

		column expected_goals
			dataType: string
			lineageTag: b3d806f9-e9fd-42e3-8790-113ac3cee25a
			summarizeBy: none
			sourceColumn: expected_goals

			annotation SummarizationSetBy = Automatic

		column expected_assists
			dataType: string
			lineageTag: 9cdfff6d-7e29-4a9d-8f06-dd6fe5c2b479
			summarizeBy: none
			sourceColumn: expected_assists

			annotation SummarizationSetBy = Automatic

		column expected_goal_involvements
			dataType: string
			lineageTag: bbca23e4-0e56-4b5d-bd05-59ad7c169928
			summarizeBy: none
			sourceColumn: expected_goal_involvements

			annotation SummarizationSetBy = Automatic

		column expected_goals_conceded
			dataType: string
			lineageTag: f85bcf26-7b52-4e44-aaf0-1c1c29361dae
			summarizeBy: none
			sourceColumn: expected_goals_conceded

			annotation SummarizationSetBy = Automatic

		column influence_rank
			dataType: string
			lineageTag: 464fad9d-c813-438b-a5ad-18efbc3b9367
			summarizeBy: none
			sourceColumn: influence_rank

			annotation SummarizationSetBy = Automatic

		column influence_rank_type
			dataType: string
			lineageTag: d480e4e4-6817-4696-98d9-a3f7f80c8c3f
			summarizeBy: none
			sourceColumn: influence_rank_type

			annotation SummarizationSetBy = Automatic

		column creativity_rank
			dataType: string
			lineageTag: e3c98a96-1ee3-48fb-841e-2b4b212a1680
			summarizeBy: none
			sourceColumn: creativity_rank

			annotation SummarizationSetBy = Automatic

		column creativity_rank_type
			dataType: string
			lineageTag: bb2f1d3a-2722-4e63-92e9-f98198aa67c7
			summarizeBy: none
			sourceColumn: creativity_rank_type

			annotation SummarizationSetBy = Automatic

		column threat_rank
			dataType: string
			lineageTag: 41262df8-aea9-4a9d-960b-72c86a1b1cc7
			summarizeBy: none
			sourceColumn: threat_rank

			annotation SummarizationSetBy = Automatic

		column threat_rank_type
			dataType: string
			lineageTag: 6c1e9868-7c3f-41cd-88d3-bad59a86a82f
			summarizeBy: none
			sourceColumn: threat_rank_type

			annotation SummarizationSetBy = Automatic

		column ict_index_rank
			dataType: string
			lineageTag: d378ccb2-a2ef-48b4-8f20-10f653b5ab67
			summarizeBy: none
			sourceColumn: ict_index_rank

			annotation SummarizationSetBy = Automatic

		column ict_index_rank_type
			dataType: string
			lineageTag: 0a5b1514-e41b-4f42-9795-19cc6c02a6d4
			summarizeBy: none
			sourceColumn: ict_index_rank_type

			annotation SummarizationSetBy = Automatic

		column corners_and_indirect_freekicks_order
			dataType: string
			lineageTag: c728f9bd-b702-4b77-8f2f-e95c511523f0
			summarizeBy: none
			sourceColumn: corners_and_indirect_freekicks_order

			annotation SummarizationSetBy = Automatic

		column corners_and_indirect_freekicks_text
			dataType: string
			lineageTag: e2f37fc2-d1ab-42f8-bd87-d6d778115921
			summarizeBy: none
			sourceColumn: corners_and_indirect_freekicks_text

			annotation SummarizationSetBy = Automatic

		column direct_freekicks_order
			dataType: string
			lineageTag: eae9fe18-2825-43b5-a0dc-bd1d37ed010e
			summarizeBy: none
			sourceColumn: direct_freekicks_order

			annotation SummarizationSetBy = Automatic

		column direct_freekicks_text
			dataType: string
			lineageTag: de6b2c99-3542-40cd-a305-751f0585c0c9
			summarizeBy: none
			sourceColumn: direct_freekicks_text

			annotation SummarizationSetBy = Automatic

		column penalties_order
			dataType: string
			lineageTag: c5097b44-f5b4-4c5e-a209-634f03e47c58
			summarizeBy: none
			sourceColumn: penalties_order

			annotation SummarizationSetBy = Automatic

		column penalties_text
			dataType: string
			lineageTag: eb17e6ed-a3ae-4df8-9a68-c5e1303ab34c
			summarizeBy: none
			sourceColumn: penalties_text

			annotation SummarizationSetBy = Automatic

		column expected_goals_per_90
			dataType: string
			lineageTag: e697e4d5-6096-4e43-88ca-455e7ebfc988
			summarizeBy: none
			sourceColumn: expected_goals_per_90

			annotation SummarizationSetBy = Automatic

		column saves_per_90
			dataType: string
			lineageTag: 25dd69a7-7131-428a-b58a-e94a50c6c388
			summarizeBy: none
			sourceColumn: saves_per_90

			annotation SummarizationSetBy = Automatic

		column expected_assists_per_90
			dataType: string
			lineageTag: f92b7df5-83a3-417a-be78-8c027ed0f0ef
			summarizeBy: none
			sourceColumn: expected_assists_per_90

			annotation SummarizationSetBy = Automatic

		column expected_goal_involvements_per_90
			dataType: string
			lineageTag: 6496ae9d-d644-4d08-843e-d898b810cdf2
			summarizeBy: none
			sourceColumn: expected_goal_involvements_per_90

			annotation SummarizationSetBy = Automatic

		column expected_goals_conceded_per_90
			dataType: string
			lineageTag: 6e852fe0-9f6f-45f0-bb99-81cde57606b3
			summarizeBy: none
			sourceColumn: expected_goals_conceded_per_90

			annotation SummarizationSetBy = Automatic

		column goals_conceded_per_90
			dataType: string
			lineageTag: af26776c-e40c-4cfd-b404-d27cbac86a11
			summarizeBy: none
			sourceColumn: goals_conceded_per_90

			annotation SummarizationSetBy = Automatic

		column now_cost_rank
			dataType: string
			lineageTag: 53157e8c-40ab-4f3e-9f9d-d42eb6093a31
			summarizeBy: none
			sourceColumn: now_cost_rank

			annotation SummarizationSetBy = Automatic

		column now_cost_rank_type
			dataType: string
			lineageTag: d80cedc7-a082-44d8-91f0-0e8cfd4507c3
			summarizeBy: none
			sourceColumn: now_cost_rank_type

			annotation SummarizationSetBy = Automatic

		column form_rank
			dataType: string
			lineageTag: 0369933b-d981-46bd-b488-2568cf8f8534
			summarizeBy: none
			sourceColumn: form_rank

			annotation SummarizationSetBy = Automatic

		column form_rank_type
			dataType: string
			lineageTag: 795bec3c-bbdb-40b3-84e0-dd5b8319876f
			summarizeBy: none
			sourceColumn: form_rank_type

			annotation SummarizationSetBy = Automatic

		column points_per_game_rank
			dataType: string
			lineageTag: 8f9a61e6-98f8-4d8b-8ffc-8e1678b212ad
			summarizeBy: none
			sourceColumn: points_per_game_rank

			annotation SummarizationSetBy = Automatic

		column points_per_game_rank_type
			dataType: string
			lineageTag: c28103a6-71cc-4f10-bc67-fb8663230f12
			summarizeBy: none
			sourceColumn: points_per_game_rank_type

			annotation SummarizationSetBy = Automatic

		column selected_rank
			dataType: string
			lineageTag: 84e72af4-614e-4358-82fa-d5fcb7a138de
			summarizeBy: none
			sourceColumn: selected_rank

			annotation SummarizationSetBy = Automatic

		column selected_rank_type
			dataType: string
			lineageTag: d7424705-58d0-4dfd-900c-db546862cfe9
			summarizeBy: none
			sourceColumn: selected_rank_type

			annotation SummarizationSetBy = Automatic

		column starts_per_90
			dataType: string
			lineageTag: 6394f577-f08b-4926-8a0a-a47428b9ac58
			summarizeBy: none
			sourceColumn: starts_per_90

			annotation SummarizationSetBy = Automatic

		column clean_sheets_per_90
			dataType: string
			lineageTag: 5fb6acf6-3673-4758-8138-fd2f797db247
			summarizeBy: none
			sourceColumn: clean_sheets_per_90

			annotation SummarizationSetBy = Automatic

		partition 'Player Summary' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    elements = Source[elements],
					    #"Converted to Table" = Table.FromList(elements, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"chance_of_playing_next_round", "chance_of_playing_this_round", "code", "cost_change_event", "cost_change_event_fall", "cost_change_start", "cost_change_start_fall", "dreamteam_count", "element_type", "ep_next", "ep_this", "event_points", "first_name", "form", "id", "in_dreamteam", "news", "news_added", "now_cost", "photo", "points_per_game", "second_name", "selected_by_percent", "special", "squad_number", "status", "team", "team_code", "total_points", "transfers_in", "transfers_in_event", "transfers_out", "transfers_out_event", "value_form", "value_season", "web_name", "minutes", "goals_scored", "assists", "clean_sheets", "goals_conceded", "own_goals", "penalties_saved", "penalties_missed", "yellow_cards", "red_cards", "saves", "bonus", "bps", "influence", "creativity", "threat", "ict_index", "starts", "expected_goals", "expected_assists", "expected_goal_involvements", "expected_goals_conceded", "influence_rank", "influence_rank_type", "creativity_rank", "creativity_rank_type", "threat_rank", "threat_rank_type", "ict_index_rank", "ict_index_rank_type", "corners_and_indirect_freekicks_order", "corners_and_indirect_freekicks_text", "direct_freekicks_order", "direct_freekicks_text", "penalties_order", "penalties_text", "expected_goals_per_90", "saves_per_90", "expected_assists_per_90", "expected_goal_involvements_per_90", "expected_goals_conceded_per_90", "goals_conceded_per_90", "now_cost_rank", "now_cost_rank_type", "form_rank", "form_rank_type", "points_per_game_rank", "points_per_game_rank_type", "selected_rank", "selected_rank_type", "starts_per_90", "clean_sheets_per_90"}, {"chance_of_playing_next_round", "chance_of_playing_this_round", "code", "cost_change_event", "cost_change_event_fall", "cost_change_start", "cost_change_start_fall", "dreamteam_count", "element_type", "ep_next", "ep_this", "event_points", "first_name", "form", "id", "in_dreamteam", "news", "news_added", "now_cost", "photo", "points_per_game", "second_name", "selected_by_percent", "special", "squad_number", "status", "team", "team_code", "total_points", "transfers_in", "transfers_in_event", "transfers_out", "transfers_out_event", "value_form", "value_season", "web_name", "minutes", "goals_scored", "assists", "clean_sheets", "goals_conceded", "own_goals", "penalties_saved", "penalties_missed", "yellow_cards", "red_cards", "saves", "bonus", "bps", "influence", "creativity", "threat", "ict_index", "starts", "expected_goals", "expected_assists", "expected_goal_involvements", "expected_goals_conceded", "influence_rank", "influence_rank_type", "creativity_rank", "creativity_rank_type", "threat_rank", "threat_rank_type", "ict_index_rank", "ict_index_rank_type", "corners_and_indirect_freekicks_order", "corners_and_indirect_freekicks_text", "direct_freekicks_order", "direct_freekicks_text", "penalties_order", "penalties_text", "expected_goals_per_90", "saves_per_90", "expected_assists_per_90", "expected_goal_involvements_per_90", "expected_goals_conceded_per_90", "goals_conceded_per_90", "now_cost_rank", "now_cost_rank_type", "form_rank", "form_rank_type", "points_per_game_rank", "points_per_game_rank_type", "selected_rank", "selected_rank_type", "starts_per_90", "clean_sheets_per_90"}),
					    #"Renamed Columns" = Table.RenameColumns(#"Expanded Column1",{{"element_type", "Position ID"}}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"Position ID", type text}}),
					    #"Sorted Rows" = Table.Sort(#"Changed Type",{{"expected_goals", Order.Descending}}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Sorted Rows",{{"selected_by_percent", type number}})
					in
					    #"Changed Type1"

		annotation PBI_NavigationStepName = Navigation

		annotation PBI_ResultType = Table




---


createOrReplace

	table 'Position (DIM)'
		lineageTag: 18dd2e2f-803e-4fae-aafe-e77fb3078724

		column 'Position ID'
			dataType: string
			lineageTag: 3a19bdc6-4942-4d41-b1f1-8bb6364a75b0
			summarizeBy: none
			sourceColumn: Position ID

			annotation SummarizationSetBy = Automatic

		column 'Position Name'
			dataType: string
			lineageTag: be230c2b-e42d-4aae-86ed-64504b4de71d
			summarizeBy: none
			sourceColumn: Position Name

			annotation SummarizationSetBy = Automatic

		partition 'Position (DIM)-128d476b-b4ff-410b-806a-c731f4c7f256' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents("https://fantasy.premierleague.com/api/bootstrap-static/")),
					    element_types = Source[element_types],
					    #"Converted to Table" = Table.FromList(element_types, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"id", "plural_name", "plural_name_short", "singular_name", "singular_name_short", "squad_select", "squad_min_play", "squad_max_play", "ui_shirt_specific", "sub_positions_locked", "element_count"}, {"id", "plural_name", "plural_name_short", "singular_name", "singular_name_short", "squad_select", "squad_min_play", "squad_max_play", "ui_shirt_specific", "sub_positions_locked", "element_count"}),
					    #"Renamed Columns" = Table.RenameColumns(#"Expanded Column1",{{"id", "Position ID"}}),
					    #"Removed Other Columns" = Table.SelectColumns(#"Renamed Columns",{"Position ID", "plural_name", "plural_name_short", "singular_name", "singular_name_short"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Removed Other Columns",{{"Position ID", type text}, {"plural_name", type text}, {"plural_name_short", type text}, {"singular_name", type text}, {"singular_name_short", type text}}),
					    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type",{{"plural_name", "Position Name"}}),
					    #"Removed Columns" = Table.RemoveColumns(#"Renamed Columns1",{"plural_name_short", "singular_name", "singular_name_short"})
					in
					    #"Removed Columns"

		annotation PBI_ResultType = Table

		annotation PBI_NavigationStepName = Navigation




---

createOrReplace

	table 'Results (DIM)'
		lineageTag: b0ce8936-6a93-4f43-884c-3c317fc8a48a

		measure 'Number of Results' = COUNTROWS('Results (DIM)')
			formatString: 0
			lineageTag: a8ef84a1-4326-44c3-a648-3580b8cc04b3

		column code
			dataType: int64
			formatString: 0
			lineageTag: 61179276-b439-490b-b4e6-f38d63135365
			summarizeBy: sum
			sourceColumn: code

			annotation SummarizationSetBy = Automatic

		column GameWeek
			dataType: int64
			formatString: 0
			lineageTag: 22bb428f-8cfe-48de-b414-a3504be8f02a
			summarizeBy: none
			sourceColumn: GameWeek

			annotation SummarizationSetBy = User

		column finished
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: 9173e79f-5806-415f-8ac1-cf7d988e4734
			summarizeBy: none
			sourceColumn: finished

			annotation SummarizationSetBy = Automatic

		column finished_provisional
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: f66aadb7-a92c-49e3-91d7-2774db3549db
			summarizeBy: none
			sourceColumn: finished_provisional

			annotation SummarizationSetBy = Automatic

		column 'Fixture ID'
			dataType: int64
			formatString: 0
			lineageTag: 736f017a-3c09-4abe-975a-e028158b81d2
			summarizeBy: none
			sourceColumn: Fixture ID

			annotation SummarizationSetBy = Automatic

		column kickoff_time
			dataType: dateTime
			formatString: General Date
			lineageTag: a19498b6-a674-45d1-9c11-ab6fd92459fa
			summarizeBy: none
			sourceColumn: kickoff_time

			variation Variation
				isDefault
				relationship: 4d6f5ca3-3a53-4bf7-b017-e3c19cb6b4e7
				defaultHierarchy: LocalDateTable_a16e547d-fb6f-4bf8-bf34-2d94cda7a2b8.'Date Hierarchy'

			annotation SummarizationSetBy = Automatic

		column team_a
			dataType: string
			lineageTag: a84fc18e-3f53-494f-a42b-61a4b0bd50aa
			summarizeBy: none
			sourceColumn: team_a

			annotation SummarizationSetBy = Automatic

		column team_a_score
			dataType: int64
			formatString: 0
			lineageTag: 63bd1720-18d3-4737-b5d1-dfa8ebdfecb3
			summarizeBy: sum
			sourceColumn: team_a_score

			annotation SummarizationSetBy = Automatic

		column team_h
			dataType: string
			lineageTag: 32ffb351-afed-4f9c-b953-c6ea9b8a318a
			summarizeBy: none
			sourceColumn: team_h

			annotation SummarizationSetBy = Automatic

		column team_h_score
			dataType: int64
			formatString: 0
			lineageTag: af8503f3-14f6-4c61-9656-1566f637f84a
			summarizeBy: sum
			sourceColumn: team_h_score

			annotation SummarizationSetBy = Automatic

		column team_h_difficulty
			dataType: int64
			formatString: 0
			lineageTag: 35a14dd9-cdfa-438d-ba62-4d5627babcc1
			summarizeBy: sum
			sourceColumn: team_h_difficulty

			annotation SummarizationSetBy = Automatic

		column team_a_difficulty
			dataType: int64
			formatString: 0
			lineageTag: 0e70d4dd-54df-47e4-8ec5-8438dedef7b9
			summarizeBy: sum
			sourceColumn: team_a_difficulty

			annotation SummarizationSetBy = Automatic

		column pulse_id
			dataType: int64
			formatString: 0
			lineageTag: 313b0ca8-3f40-44a0-827c-a3c8e614a2a5
			summarizeBy: sum
			sourceColumn: pulse_id

			annotation SummarizationSetBy = Automatic

		column Result
			dataType: string
			lineageTag: 090183b8-0c05-4de7-b512-fb52197b565c
			summarizeBy: none
			sourceColumn: Result

			annotation SummarizationSetBy = Automatic

		column 'Expected Result'
			dataType: string
			lineageTag: 399b25a6-5647-42f3-bed5-3b894d13da4f
			summarizeBy: none
			sourceColumn: Expected Result

			annotation SummarizationSetBy = Automatic

		column 'Exceptional Result'
			dataType: boolean
			formatString: """TRUE"";""TRUE"";""FALSE"""
			lineageTag: 5b06904d-31c6-403b-80ad-0cf2e43b0bd6
			summarizeBy: none
			sourceColumn: Exceptional Result

			annotation SummarizationSetBy = Automatic

		column 'kickoff_time - Copy'
			dataType: dateTime
			formatString: Long Time
			lineageTag: 79fbba47-99a8-4e14-ae3d-2794c4d71e2a
			summarizeBy: none
			sourceColumn: kickoff_time - Copy

			annotation SummarizationSetBy = Automatic

			annotation UnderlyingDateTimeDataType = Time

		partition 'Results (DIM)-72dbf123-0342-4565-8aef-3c6b0923cb85' = m
			mode: import
			source =
					let
					    Source = Json.Document(Web.Contents(" https://fantasy.premierleague.com/api/fixtures/")),
					    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
					    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"code", "event", "finished", "finished_provisional", "id", "kickoff_time", "minutes", "provisional_start_time", "started", "team_a", "team_a_score", "team_h", "team_h_score", "stats", "team_h_difficulty", "team_a_difficulty", "pulse_id"}, {"code", "event", "finished", "finished_provisional", "id", "kickoff_time", "minutes", "provisional_start_time", "started", "team_a", "team_a_score", "team_h", "team_h_score", "stats", "team_h_difficulty", "team_a_difficulty", "pulse_id"}),
					    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Column1",{{"id", type text}}),
					    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"id", "Fixture ID"}, {"event", "GameWeek"}}),
					    #"Removed Other Columns" = Table.SelectColumns(#"Renamed Columns",{"code", "finished", "finished_provisional", "Fixture ID", "GameWeek", "kickoff_time", "pulse_id", "stats", "team_a", "team_a_difficulty", "team_a_score", "team_h", "team_h_difficulty", "team_h_score"}),
					    #"Removed Columns" = Table.RemoveColumns(#"Removed Other Columns",{"stats"}),
					    #"Changed Type1" = Table.TransformColumnTypes(#"Removed Columns",{{"kickoff_time", type datetime}}),
					    #"Filtered Rows" = Table.SelectRows(#"Changed Type1", each ([finished] = true)),
					    #"Changed Type4" = Table.TransformColumnTypes(#"Filtered Rows",{{"finished", type logical}}),
					    #"Changed Type2" = Table.TransformColumnTypes(#"Changed Type4",{{"finished", type logical}, {"code", Int64.Type}, {"finished_provisional", type logical}, {"Fixture ID", Int64.Type}, {"GameWeek", Int64.Type}, {"pulse_id", Int64.Type}, {"team_a", type text}, {"team_a_difficulty", Int64.Type}, {"team_a_score", Int64.Type}, {"team_h", type text}, {"team_h_difficulty", Int64.Type}, {"team_h_score", Int64.Type}}),
					    #"Added Conditional Column" = Table.AddColumn(#"Changed Type2", "Result", each if [team_a_score] = [team_h_score] then "Draw" else if [team_a_score] > [team_h_score] then "Away Win" else if [team_a_score] < [team_h_score] then "Home Win" else null),
					    #"Added Conditional Column1" = Table.AddColumn(#"Added Conditional Column", "Expected Result", each if [team_a_difficulty] > [team_h_difficulty] then "Away Win" else if [team_a_difficulty] < [team_h_difficulty] then "Home Win" else if [team_a_difficulty] = [team_h_difficulty] then "Draw" else null),
					    #"Changed Type3" = Table.TransformColumnTypes(#"Added Conditional Column1",{{"Expected Result", type text}}),
					    #"Added Conditional Column2" = Table.AddColumn(#"Changed Type3", "Exceptional Result", each if [Result] <> [Expected Result] then true else false),
					    #"Changed Type5" = Table.TransformColumnTypes(#"Added Conditional Column2",{{"Exceptional Result", type logical}}),
					    #"Duplicated Column" = Table.DuplicateColumn(#"Changed Type5", "kickoff_time", "kickoff_time - Copy"),
					    #"Changed Type6" = Table.TransformColumnTypes(#"Duplicated Column",{{"kickoff_time - Copy", type time}})
					in
					    #"Changed Type6"

		annotation PBI_ResultType = Table

		annotation PBI_NavigationStepName = Navigation




---


Create a Series of CREATE staements that builds this model in SQL Database





