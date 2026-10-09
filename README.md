package com.example.foodrescueapp;

import android.content.res.ColorStateList;
import android.graphics.Color;
import android.graphics.Typeface;
import android.graphics.drawable.GradientDrawable;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;

import java.io.File;
import java.util.ArrayList;

public class MyClaimsActivity extends AppCompatActivity {

    private LinearLayout layoutClaimsList;
    private LinearLayout layoutEmptyClaims;
    private TextView tvNoClaims;
    private Button btnBack;

    private final int DARK_GREEN =
            Color.parseColor("#1F5D42");

    private final int TEXT_COLOR =
            Color.parseColor("#263B2C");

    private final int SECONDARY_COLOR =
            Color.parseColor("#718075");

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_my_claims);

        // =========================================
        // INITIALIZE LOCAL STORAGE
        // =========================================

        DonationManager.initialize(
                getApplicationContext()
        );

        // =========================================
        // CONNECT JAVA WITH XML
        // =========================================

        layoutClaimsList =
                findViewById(R.id.layoutClaimsList);

        layoutEmptyClaims =
                findViewById(R.id.layoutEmptyClaims);

        tvNoClaims =
                findViewById(R.id.tvNoClaims);

        btnBack =
                findViewById(R.id.btnBack);

        btnBack.setOnClickListener(v -> finish());

        loadClaimedFood();
    }

    @Override
    protected void onResume() {

        super.onResume();

        if (layoutClaimsList != null) {
            loadClaimedFood();
        }
    }

    // =========================================
    // LOAD CLAIMED FOOD
    // =========================================

    private void loadClaimedFood() {

        FirebaseUser currentUser =
                FirebaseAuth.getInstance()
                        .getCurrentUser();

        if (currentUser == null) {

            layoutClaimsList.removeAllViews();

            layoutEmptyClaims.setVisibility(
                    View.VISIBLE
            );

            tvNoClaims.setText(
                    "Please login to view your claims."
            );

            return;
        }

        String currentUserId =
                currentUser.getUid();

        DonationManager.loadDonations(
                new DonationManager.LoadCallback() {

                    @Override
                    public void onSuccess(
                            ArrayList<Donation> donations
                    ) {

                        displayClaimedFood(
                                donations,
                                currentUserId
                        );
                    }

                    @Override
                    public void onFailure(Exception e) {

                        Toast.makeText(
                                MyClaimsActivity.this,
                                "Failed to load claims: "
                                        + e.getMessage(),
                                Toast.LENGTH_LONG
                        ).show();
                    }
                }
        );
    }

    // =========================================
    // DISPLAY CLAIMED FOOD
    // =========================================

    private void displayClaimedFood(
            ArrayList<Donation> donations,
            String currentUserId
    ) {

        layoutClaimsList.removeAllViews();

        boolean hasClaims = false;

        for (Donation donation : donations) {

            boolean isClaimed =
                    "Claimed".equalsIgnoreCase(
                            donation.getStatus()
                    );

            boolean belongsToRecipient =
                    currentUserId.equals(
                            donation.getClaimedBy()
                    );

            if (isClaimed && belongsToRecipient) {

                createClaimCard(donation);

                hasClaims = true;
            }
        }

        if (hasClaims) {

            layoutEmptyClaims.setVisibility(
                    View.GONE
            );

        } else {

            layoutEmptyClaims.setVisibility(
                    View.VISIBLE
            );

            tvNoClaims.setText(
                    "You have not claimed any food yet."
            );
        }
    }

    // =========================================
    // CREATE CLAIM CARD
    // =========================================

    private void createClaimCard(Donation donation) {

        LinearLayout card =
                new LinearLayout(this);

        card.setOrientation(
                LinearLayout.VERTICAL
        );

        card.setPadding(
                dp(18),
                dp(18),
                dp(18),
                dp(18)
        );

        GradientDrawable cardBackground =
                new GradientDrawable();

        cardBackground.setColor(
                Color.WHITE
        );

        cardBackground.setCornerRadius(
                dp(20)
        );

        cardBackground.setStroke(
                dp(1),
                Color.parseColor("#E2EBDD")
        );

        card.setBackground(cardBackground);

        card.setElevation(dp(3));

        LinearLayout.LayoutParams cardParams =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT
                );

        cardParams.setMargins(
                0,
                dp(5),
                0,
                dp(18)
        );

        layoutClaimsList.addView(
                card,
                cardParams
        );

        // =========================================
        // FOOD IMAGE
        // =========================================

        String imagePath =
                donation.getImagePath();

        if (imagePath != null
                && !imagePath.trim().isEmpty()) {

            File imageFile =
                    new File(imagePath);

            if (imageFile.exists()) {

                ImageView foodImage =
                        new ImageView(this);

                LinearLayout.LayoutParams imageParams =
                        new LinearLayout.LayoutParams(
                                LinearLayout.LayoutParams.MATCH_PARENT,
                                dp(190)
                        );

                imageParams.setMargins(
                        0,
                        0,
                        0,
                        dp(12)
                );

                foodImage.setLayoutParams(
                        imageParams
                );

                foodImage.setScaleType(
                        ImageView.ScaleType.CENTER_CROP
                );

                foodImage.setImageURI(
                        Uri.fromFile(imageFile)
                );

                foodImage.setContentDescription(
                        donation.getFoodName()
                );

                GradientDrawable imageBackground =
                        new GradientDrawable();

                imageBackground.setColor(
                        Color.parseColor("#F6F7F0")
                );

                imageBackground.setCornerRadius(
                        dp(14)
                );

                foodImage.setBackground(
                        imageBackground
                );

                foodImage.setClipToOutline(true);

                card.addView(foodImage);
            }
        }

        // =========================================
        // FOOD NAME
        // =========================================

        TextView foodName =
                new TextView(this);

        foodName.setText(
                "🍱  " + donation.getFoodName()
        );

        foodName.setTextSize(20);

        foodName.setTextColor(
                DARK_GREEN
        );

        foodName.setTypeface(
                null,
                Typeface.BOLD
        );

        LinearLayout.LayoutParams nameParams =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT
                );

        nameParams.setMargins(
                0,
                dp(4),
                0,
                dp(8)
        );

        card.addView(
                foodName,
                nameParams
        );

        // =========================================
        // DIVIDER
        // =========================================

        View divider =
                new View(this);

        divider.setBackgroundColor(
                Color.parseColor("#E2EBDD")
        );

        LinearLayout.LayoutParams dividerParams =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT,
                        dp(1)
                );

        dividerParams.setMargins(
                0,
                dp(8),
                0,
                dp(12)
        );

        card.addView(
                divider,
                dividerParams
        );

        // =========================================
        // FOOD INFORMATION
        // =========================================

        card.addView(
                createInfoRow(
                        "🏷️  Category",
                        donation.getCategory()
                )
        );

        card.addView(
                createInfoRow(
                        "📦  Quantity",
                        donation.getQuantity()
                )
        );

        card.addView(
                createInfoRow(
                        "📍  Collection Area",
                        donation.getCollectionArea()
                )
        );

        card.addView(
                createInfoRow(
                        "📅  Pickup Date",
                        donation.getPickupDate()
                )
        );

        card.addView(
                createInfoRow(
                        "🕒  Pickup Time",
                        donation.getPickupTime()
                )
        );

        card.addView(
                createInfoRow(
                        "📝  Description",
                        donation.getDescription()
                )
        );

        // =========================================
        // CLAIM STATUS
        // =========================================

        TextView status =
                new TextView(this);

        status.setText("●  Claimed");

        status.setTextSize(14);

        status.setTypeface(
                null,
                Typeface.BOLD
        );

        status.setTextColor(
                Color.parseColor("#A05C18")
        );

        status.setPadding(
                dp(12),
                dp(10),
                dp(12),
                dp(10)
        );

        GradientDrawable statusBackground =
                new GradientDrawable();

        statusBackground.setColor(
                Color.parseColor("#FFF1DB")
        );

        statusBackground.setCornerRadius(
                dp(12)
        );

        status.setBackground(
                statusBackground
        );

        LinearLayout.LayoutParams statusParams =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.WRAP_CONTENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT
                );

        statusParams.setMargins(
                0,
                dp(16),
                0,
                dp(4)
        );

        card.addView(
                status,
                statusParams
        );

        // =========================================
        // UNCLAIM BUTTON
        // =========================================

        Button btnUnclaim =
                new Button(this);

        btnUnclaim.setText(
                "Unclaim Food"
        );

        btnUnclaim.setTextColor(
                Color.WHITE
        );

        btnUnclaim.setTextSize(15);

        btnUnclaim.setTypeface(
                null,
                Typeface.BOLD
        );

        btnUnclaim.setBackgroundTintList(
                ColorStateList.valueOf(
                        Color.parseColor("#C44747")
                )
        );

        LinearLayout.LayoutParams unclaimParams =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT,
                        dp(52)
                );

        unclaimParams.setMargins(
                0,
                dp(20),
                0,
                0
        );

        card.addView(
                btnUnclaim,
                unclaimParams
        );

        // =========================================
        // UNCLAIM FUNCTION
        // =========================================

        btnUnclaim.setOnClickListener(v -> {

            new AlertDialog.Builder(
                    MyClaimsActivity.this
            )
                    .setTitle("Unclaim Food")
                    .setMessage(
                            "Are you sure you want to unclaim \""
                                    + donation.getFoodName()
                                    + "\"?"
                    )
                    .setNegativeButton(
                            "Cancel",
                            (dialog, which) ->
                                    dialog.dismiss()
                    )
                    .setPositiveButton(
                            "Unclaim",
                            (dialog, which) -> {

                                btnUnclaim.setEnabled(false);

                                DonationManager.updateStatus(
                                        donation.getDocumentId(),
                                        "Available",
                                        new DonationManager.DonationCallback() {

                                            @Override
                                            public void onSuccess() {

                                                Toast.makeText(
                                                        MyClaimsActivity.this,
                                                        "Food unclaimed successfully!",
                                                        Toast.LENGTH_SHORT
                                                ).show();

                                                loadClaimedFood();
                                            }

                                            @Override
                                            public void onFailure(
                                                    Exception e
                                            ) {

                                                btnUnclaim.setEnabled(true);

                                                Toast.makeText(
                                                        MyClaimsActivity.this,
                                                        "Unclaim failed: "
                                                                + e.getMessage(),
                                                        Toast.LENGTH_LONG
                                                ).show();
                                            }
                                        }
                                );
                            }
                    )
                    .show();
        });
    }

    // =========================================
    // CREATE INFORMATION ROW
    // =========================================

    private LinearLayout createInfoRow(
            String label,
            String value
    ) {

        LinearLayout row =
                new LinearLayout(this);

        row.setOrientation(
                LinearLayout.VERTICAL
        );

        row.setPadding(
                0,
                dp(7),
                0,
                dp(7)
        );

        TextView labelText =
                new TextView(this);

        labelText.setText(label);

        labelText.setTextSize(13);

        labelText.setTextColor(
                SECONDARY_COLOR
        );

        labelText.setTypeface(
                null,
                Typeface.BOLD
        );

        row.addView(labelText);

        TextView valueText =
                new TextView(this);

        valueText.setText(
                value == null
                        || value.trim().isEmpty()
                        ? "-"
                        : value
        );

        valueText.setTextSize(15);

        valueText.setTextColor(
                TEXT_COLOR
        );

        LinearLayout.LayoutParams valueParams =
                new LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT
                );

        valueParams.setMargins(
                dp(27),
                dp(4),
                0,
                0
        );

        row.addView(
                valueText,
                valueParams
        );

        return row;
    }

    // =========================================
    // DP CONVERTER
    // =========================================

    private int dp(int value) {

        return (int) (
                value *
                        getResources()
                                .getDisplayMetrics()
                                .density
                        + 0.5f
        );
    }
}
